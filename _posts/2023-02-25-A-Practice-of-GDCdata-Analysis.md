---
layout: post
title: R analysis for GDC data
date: 2021-09-02
Author: Siyuan Li
toc: true
tags: [R, notes]
comments: true
--- 
#1GDC数据获取
##1.1数据下载
###1.1.1安装gdc client
以及准备好库和目录

    if(FALSE){
      options(stringsAsFactors = F)
      library(stringr)
      cancer_type="TCGA-CHOL"
      if(!dir.exists("clinical"))dir.create("clinical")
      if(!dir.exists("expdata"))dir.create("expdata")
      dir()
    }
###1.1.2在gdc网站下载数据文件
case-file-manifest
3terminal启动gdc-client
    
    #在terminl运行以下
      #./gdc-client download -m filename.txt -d clinical
      #./gdc-client download -m filename.txt -d expdata
##1.2临床信息数据整理
0单个文件读取整理（省略
1批量读取
    
    #批量读取clinical下 不分文件层级 *任意文件名 $以xml结尾的文件
    if(FALSE){
      xmls=dir("clinical/",pattern = "*.xml$",recursive = T)
    }
2写个函数用于批量整理
    
    if(FALSE)
    {
      td=function(x){
      #读取xml文件
      result=xmlParse(file.path("clinical/",x))
      #获取文件内节点数
      noderoot=xmlroot(result)
      #把第二个节点转换成数据框
      resultdataframe=xmlToDataFrame(noderoot[2])
      #把行转置成列
      return(t(resultdataframe))
    }
    }
3批量整理
    
    if(FALSE){
      #处理 成一个列表
    cl=lapply(xmls,td)
    #列表化为一列是一个病人记录的矩阵,再转置成一行一条记录的矩阵,再转成数据框
    cl_df=data.frame(t(do.call(cbind,cl)))
    }
##1.3expdata数据整理
同上原理,代码类似
    
    if(FALSE){
      exps=dir("expdata/",pattern = "*.htseq.counts.gz$",recursive = T)
      ex=function(x){
         #第一列作为行名,以tab键作为分隔符号
      result=read.table(file.path("expdata/",x),row.names = 1,sep = "\t")
      return(result)}
      exp=lapply(exps, ex)
      exp<-do.call(cbind,exp)
    }
这时候我们发现exp数据框是没有列名的 没有列名就意味着临床数据文件和expdata对应不了

##1.4数据对应
GDC网站上加购所有数据 然后下载metadata 得到一个json文件
    
    if(FALSE){
      meta=jsonlite::fromJSON("filename")
      ids=meta$associated
      ID=sapply(ids,function(x){x[,1]})
      #这个案例的特点是有用的信息在ids[[1]],所以不做特殊处理
      #ids[[1]]的信息是在第一列,所以[,1]
      file2id=data.frame(file_name=meta$file_name,ID=ID)
      #count_filea不是全部都要,只要/后面的
      exps2=
        stringr::str_split(exps,"/",simplify=T)[,2]
      file2id = file2id[match(exps2,file2id$file_name),]
      colnames(exp) = file2id$ID
    }
##1.5去除样本表达量过低的基因
    if(FALSE){
      exp=exp[apply(exp,1,function(x) sum(x>1)>9),]
      #9是自定义的 由于一共有45个样本 所以自定义要20%以上表达才保留
    }
#2差异分析
##2.1预处理
TCGA1415位代表了是否是肿瘤 <10 tumor >=10 normal
    
    if(FALSE){
      table(str_sub(colnames(exp),14,15))
      group_list = ifelse(as.numeric(str_sub(colnames(exp),14,15)) < 10,'tumor' , 'normal')
      group_list = factor(group_list,levels =c("normal","tumor"))
      table(group_list)
      save(exp,clinical,group_list,cancer_type,
           file=paste0(cancer_type,"gdc.Rdata"))
    }
##2.2limma/edgeR/DESeq2
分析的部分本来有一百多行的代码和笔记 Rstudio出问题了全没了 气得。。 不想补了 找了XJ Sun放在GitHub上的代码当作替代

    if(F)
    {
      ##' get_deg
    ##'
    ##' do differential analysis according to expression set and group information
    ##'
    ##' @inheritParams draw_pca
    ##' @inheritParams draw_volcano
    ##' @param entriz whether convert symbols to entriz ids
    ##' @param ids a data.frame with 2 columns,including probe_id and symbol
    ##' @return a deg data.frame with 10 columns
    ##' @author Xiaojie Sun
    ##' @importFrom limma lmFit
    ##' @importFrom limma eBayes
    ##' @importFrom limma topTable
    ##' @importFrom clusterProfiler bitr
    ##' @importFrom dplyr mutate
    ##' @importFrom dplyr inner_join
    ##' @export
    ##' @examples
    ##' \donttest{gse = "GSE42872"
    ##' geo = geo_download(gse,destdir=tempdir(),by_annopbrobe = FALSE)
    ##' Group = rep(c("control","treat"),each = 3)
    ##' Group = factor(Group)
    ##' find_anno(geo$gpl)
    ##' ids <- AnnoProbe::idmap(geo$gpl,destdir = tempdir())
    ##' deg = get_deg(geo$exp,Group,ids)
    ##' head(deg)
    ##' }
    ##' @seealso
    ##' \code{\link{multi_deg}};\code{\link{get_deg_all}}
    
    get_deg <- function(exp,
                        group_list,
                        ids,
                        logFC_cutoff=1,
                        pvalue_cutoff=0.05,
                        adjust = FALSE,
                        entriz = TRUE) {
      p3 <- is.factor(group_list)
      if(!p3) {
        group_list = factor(group_list)
        warning("group_list was covert to factor")
      }
      if(nlevels(group_list)==2){
        if(ncol(exp)!=length(group_list))stop("wrong group_list or exp")
        if(ncol(ids)!=2)stop("wrong ids pramater,it should be a data.frame with probe_id and symbol")
        colnames(ids) = c("probe_id","symbol")
    
        design=stats::model.matrix(~group_list)
        fit=lmFit(exp,design)
        fit=eBayes(fit)
        deg=topTable(fit,coef=2,number = Inf)
    
        if("ID" %in% colnames(deg)){
          deg <- mutate(deg,probe_id=deg$ID)
        }else{
          deg <- mutate(deg,probe_id=rownames(deg))
        }
        ids = ids[!duplicated(ids$symbol),]
        deg <- inner_join(deg,ids,by="probe_id")
        if(adjust){
          k1 = (deg$adj.P.Val < pvalue_cutoff)&(deg$logFC < -logFC_cutoff)
          k2 = (deg$adj.P.Val < pvalue_cutoff)&(deg$logFC > logFC_cutoff)
        }else{
          k1 = (deg$P.Value < pvalue_cutoff)&(deg$logFC < -logFC_cutoff)
          k2 = (deg$P.Value < pvalue_cutoff)&(deg$logFC > logFC_cutoff)
        }
    
        change = ifelse(k1,
                        "down",
                        ifelse(k2,
                               "up",
                               "stable"))
        deg <- mutate(deg,change)
    
        if(entriz){
          s2e <- bitr(deg$symbol,
                      fromType = "SYMBOL",
                      toType = "ENTREZID",
                      OrgDb = org.Hs.eg.db::org.Hs.eg.db)
    
          deg <- inner_join(deg,s2e,by=c("symbol"="SYMBOL"))
          deg <- deg[!duplicated(deg$symbol),]
        }
      }else{
        deg <-multi_deg(exp = exp,
                        group_list = group_list,
                        ids = ids,
                        logFC_cutoff = logFC_cutoff,
                        pvalue_cutoff = pvalue_cutoff,
                        adjust = adjust,
                        entriz = entriz)
      }
      return(deg)
    }
    }
#3生存分析
##3.1KaplanMeier
KM生存分析是描述性的
    
    if(FALSE){
      #描述性生存分析
      sfit<-survfit(Surv(time, event)~gender, data=meta)
      #time总生存期 event终结事件 #gender是分组依据 meta是临床数据
    }
##3.2Cox回归
Cox回归是回归模型,没有直接使用生存时间,而是使用了风险比(hazardratio)作为因变量。该模型不用于估计生存率,而是用于因素分析,也就是找到某一个危险因素对结局事件发生的贡献度。
风险比 (hazard ratio)
当HR>1时,说明研究对象是一个危险因素
当HR<1时,说明研究对象是一个保护因素
当HR=1时,说明研究对象对生存时间不起作用

log-rank和cox都可以给每个基因计算一个p值,计算其对生存的影响

###3.2.1前期准备
    if(FALSE){
      knitr::opts_chunk$set(collapse = TRUE,
                            comment ="#>")
      knitr::opts_chunk$set(fig.width = 6,fig.height =6,collapse = TRUE)
      knitr::opts_chunk$set(message = FALSE)
      
      
    }
###3.2.2整理数据
    if(FALSE){
      rm(list=Is(0))
      options(stringsAsFactors = F)
      Load("TCGA-CHOLgdc,Rdata")
      library(stringr)
    #clinical通常有几十列,选出其中有用的几列
      tmp = data.frame(colnames(clinical))
      clinical = clinical[,c(
      "bcr_patient_barcode",
      "vital_status'days_to_death",
      "days_to_last_followup'race_list",
      "days_to_birth",
      "gender",
      "stage_event"
      )]
      #其实days_to_last_followup应该是找age_at_initial_pathologic_diagnosis,这表格里没有,用days_to_birth计算一下年龄,暂且替代
      dim(clinical)
    #rownames(clinical) <- NULL
      rownames(clinical) <- clinical$bcr_patient_barcode
      clinical[1:4,1:4]
      
      meta = clinical
      exprSet=exp[,group_list=='tumor']
      #简化meta的列名
      colnames(meta)=c('ID','event','death','last_followup','race',' age', 'gender','stage')
      #调整meta的ID顺序与exprSet列名一致
      meta=meta[match(str_sub(colnames(exprSet),1,12),meta$ID),]
      all(meta$ID==str_sub(colnames(exprSet),1,12))
                
    }
###3.2.3整理生存分析的输入数据
由随访时间和死亡时间计算生存时间(月)

    if(FALSE){
      is.empty.chr = function(x){
        ifelse(stringr::str_length(x)==0,T,F)}
      is.empty.chr(meta[1,4])
      meta[,3][is.empty.chr(meta[,3])]=0
      meta[,4][is.empty.chr(meta[,4])]=0
      meta$time=(as.numeric(meta[,3])+as.numeric(meta[,4]))/30
      
    }
根据生死定义event,活着是0,死的是1
    
    if(FALSE){
      meta$event=ifelse(meta$event=='Alive',0,1)
    }
年龄和年龄分组
    
    if(FALSE){
      meta$age=ceiling(abs(as.numeric(meta$age))/365)
      meta$age_group=ifelse(meta$age>median(meta$age),'older','younger')
    
    }
stage
    
    if(FALSE){
      library(stringr)
      meta$stagetmp = str_split(meta$stage," ",simplify = T)[,2]
      table(str_count(tmp,"T"))
      str_locate(tmp,"T")[,1]
      tmp = str_sub(tmp,1,str_locate(tmp,"T")[,1]-1)
      table(tmp)
      meta$stage = tmp
    }
race
    
    if(FALSE){
      table(meta$race)
      save(meta,exprSet,cancer_type,
           file =paste0(cancer_type,"sur_model.Rdata"))
    }
##3.3绘图
###3.3.1生存分析绘图
    if(FALSE){
      rm(list = Is())
    Load("TCGA-CHOLsur_model.Rdata")
    library(survival)
    library(survminer)
    #年龄
    sfit <- survfit(Surv(time,event)~age_group,data=meta)
    ggsurvplot(sfit,conf.int=F,pval=TRUE)
    
    ggsurvplot(sfit,palette = c("#E7B800","#2E9FDF"),
               risk.table =TRUE,pyal =TRUE,conf.int =TRUE,
               xlab ="Time in months",
               ggtheme =theme_light(),ncensor.plot = TRUE)
    #性别年龄
    sfit1=survfit(Surv(time,event)~gender,data=meta)
    sfit2=survfit(Surv(time,event)~age_group,data=meta)
    splots <- list()
    splots[[1]] <- ggsurvplot(sfit1,pval =TRUE,data =meta,risk.table = TRUE)
    splots[[2]] <- ggsurvplot(sfit2,pval =TRUE,data =meta,risk.table = TRUE)
    arrange_ggsurvplots(splots,print = TRUE,ncol = 2,nrow = 1,risk.table.height = 0.4)
    dev.off()
    #单个基因
    g = rownames(exprSet)[1]
    meta$gene = ifelse(as,integer(exprSet[g,])>
    median(as.integer(exprSet[g,])),"high","low")
    sfit1=survfit(Surv(time,event)~gene,data=meta)
    ggsurvplot(sfit1,pval =TRUE,data = meta,risk.table=TRUE)
    #多个基因
    gs=rownames(exprSet)[1:4]
    splots <- lapply(gs,function(g){
    meta$gene=ifelse(as,integer(exprSet[g,])>median(as.integer(exprSet[g,])),"high","low")
    sfit1=survfit(Surv(time,event)~gene,data=meta)
    ggsurvplot(sfit1,pval =TRUE,data = meta,risk.table = TRUE)})
    arrange_ggsurvplots(splots,print = TRUE,ncol = 2,nrow = 2,risk.table.height = 0.4
    )
      
    }
###3.3.2Logrank批量生存分析
    if(FALSE){
      
    logrankfile = paste(cancer_type,"log_rank_p.Rdata")
    if(!file.exists(logrankfile)){
    mySurv=with(meta,Surv(time,event))
    log_rank_p <- apply(exprSet ,1 ,function(gene){
    # gene=exprSet[1,]
    meta$group=ifelse(gene>median(gene),"high","low")
    data.survdiff=survdiff(mySurv~group,data=meta)
    p.val = 1 - pchisq(data.survdiff$chisq,
    length(data.survdiff$n) - 1)
    return(p.val)
    })
    log_rank_p=sort(log_rank_p)
    save(log_rank_p,file = logrankfile)
    }
    load(logrankfile)
    table(log_rank_p<0.01)
    table(log_rank_p<0.05)
    }
###3.3.3COX生存分析批量绘图
    if(FALSE){
      
    coxfile = paste0(cancer_type,"cox.Rdata")
    if(!file.exists(coxfile)){
      mySurv=with(meta,Surv(time,event))
      cox_results <-apply(exprSet,1,function(gene){
        # gene= as.numeric(exprSet[1,])
        group=ifelse(gene>median(gene),"high","low")
        survival_dat <-data.frame(group=group,
                                  stage=meta$stage,
                                  age=meta$age,
                                  gender=meta$gender,
                                  gene= gene,
                                  stringsAsFactors = F)
        #表达量转为二分类变量
        m=coxph(mySurv ~ gender + age + stage + group,datasurvival_dat)
        #m=coxph(mySurv ~Group,data = survival_dat)
        #也可直接使用连续型变量
        #m = coxph(mySurv ~ gene,data = survival_dat)
        beta <- coef(m)
        se <- sqrt(diag(vcov(m)))
        HR <- exp(beta)
        HRse <- HR*se
    
      #summary(m)
        tmp <- round(cbind(coef = beta, se = se, z = beta/se,
                           p =1-pchisq((beta/se)^2,1),
                           HR = HR,HRse = HRse,
                           HRZ = (HR - 1) / HRse,
                           HRp = 1 - pchisq(((HR-1)/HRse)/2,1),
                           HRCILL = exp(beta - qnorm(.975,0,1)* se),
                           HRCIUL = exp(beta + gnorm(.975,0,1)* se)),3) 
     return(tmp["grouplow",])
     #return(tmp['gene',]) #连续型变量
        
    })
      
      cox_results=t(cox_results)
      save(cox_results,file = coxfile)
    }
    load(coxfile)
    table(cox_results[,4]<0.01)
    table(cox_results[,4]<0.05)
    
    Ir = names(log_rank_p)[log_rank_p<0.01]
    cox = rownames(cox_results)[cox_results[,4]<0.01]
    length(intersect(lr,cox))
    }
