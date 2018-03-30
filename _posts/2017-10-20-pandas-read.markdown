---
layout: post
category: "read"
title:  "Pandas笔记"
---

创建序列  
pd.Series([u,v,w,x,y,z])

日期序列  
pd.date_range(date,periods=num)

创建dataframe  
pd.DataFrame(matixdata，index=rownames，columns=colnames)  
pd.DataFrame(dictionarydata)

dataframe行名称 df.index  
dataframe列名称 df.columns  
dataframe值 df.values   
dataframe值类型 df.dtype  
dataframe描述 df.describe()

dataframe转置(行列互转) df.T  
dataframe排序  
df.sort_index(axis=0/1,ascending=False/True)  
df.sort_values(by=rowname/colname)  

从标签选择 df.loc[rowname,colname]  
从位置选择 df.iloc[rowindex,colindex]

丢弃NaN的行/列 df.dropna(axis=0/1,how='any/all')  
填充NaN的数据 df.fillna(value=value)  
判断是否缺失 df.isnull()  

导入导出数据  
pd.read_csv(csvfile)  
data.to_pickle(pklfile)

合并数据    
pd.concat([df1,df2,df3],axis=0/1,ignore_index=True)  
pd.concat([df1,df2,df3],join="inner/outer",ignore_index=True)  
pd.concat([df1,df2,df3],join_axis=[df1.index],ignore_index=True)





