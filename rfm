# -*- coding: utf-8 -*-
"""
Created on Sun Jan 12 17:05:58 2020

@author: Mario
R（最近一次购买距今多少天)，F（购买了多少次）以及M（平均或者累计购买金额）。
"""
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_excel('PYTHON-RFM实战数据.xlsx')
#数据清洗 
#df0 = df.loc[df['订单状态'] == '交易成功',:]
df = df[df['订单状态'] == '交易成功']
df = df[['买家昵称','付款日期','实付金额']]
#R
r = df.groupby('买家昵称')['付款日期'].max().reset_index()
r['R'] = (pd.to_datetime('2020-1-12') - r['付款日期']).dt.days
r = r[['买家昵称','R']]
#F
df['日期标签'] = df['付款日期'].astype(str).str[:10]
dup_f = df.groupby(['买家昵称','日期标签'])['付款日期'].count().reset_index()
f = dup_f.groupby('买家昵称')['付款日期'].count().reset_index()
f.columns = ['买家昵称','F']
#M
sum_m = df.groupby('买家昵称')['实付金额'].sum().reset_index()
sum_m.columns = ['买家昵称','总支付金额']
com_m = pd.merge(sum_m,f,left_on = '买家昵称',right_on = '买家昵称',how = 'inner')
com_m['M'] = com_m['总支付金额'] / com_m['F']

rfm = pd.merge(r,com_m,left_on = '买家昵称',right_on = '买家昵称',how = 'inner')
rfm = rfm[['买家昵称','R','F','M']]
#维度打分
bins = [150,200,250,300,350,100000]
labels = [5,4,3,2,1]
rfm['R-score'] = pd.cut(rfm['R'],bins = bins,labels = labels,right = False).astype(float)
bins = [1,2,3,4,5,100000]
labels = [1,2,3,4,5]
rfm['F-score'] = pd.cut(rfm['F'],bins = bins,labels = labels,right = False).astype(float)
bins = [0,50,100,150,200,100000]
labels = [1,2,3,4,5]
rfm['M-score'] = pd.cut(rfm['M'],bins = bins,labels = labels,right = False).astype(float)
rfm['R是否大于平均值'] = (rfm['R-score'] > rfm['R-score'].mean()) * 1
rfm['F是否大于平均值'] = (rfm['F-score'] > rfm['F-score'].mean()) * 1
rfm['M是否大于平均值'] = (rfm['M-score'] > rfm['M-score'].mean()) * 1
#客户分层
rfm['人群数值'] = (rfm['R是否大于平均值'] * 100) + (rfm['F是否大于平均值'] * 10) + (rfm['M是否大于平均值'] * 1)
def transform_label(x):
    if x == 111:
        label = '重要价值客户'
    elif x == 110:
        label = '消费潜力客户'
    elif x == 101:
        label = '频次深耕客户'
    elif x == 100:
        label = '新客户'
    elif x == 11:
        label = '重要价值流失预警客户'
    elif x == 10:
        label = '一般客户'
    elif x == 1:
        label = '高消费唤回客户'
    elif x == 0:
        label = '流失客户'
    return label
rfm['客户类型'] = rfm['人群数值'].apply(transform_label)
#结果分析
rfm_count = rfm.groupby('客户类型')['买家昵称'].count().reset_index()
rfm_count.columns = ['客户类型','客户人数']
rfm_count['人数占比'] = rfm_count['客户人数'] / rfm_count['客户人数'].sum()
rfm['购买总金额'] = rfm['F'] * rfm['M']
rfm_count['消费金额'] = list(rfm.groupby('客户类型')['购买总金额'].sum())
rfm_count['金额占比'] = rfm_count['消费金额'] / rfm_count['消费金额'].sum()
#可视化
def visual_show(x,y1,y2,name):
    plt.rcParams['font.sans-serif'] = ['KaiTi']
    plt.figure()
    plt.bar(x,y2)
#    plt.plot(x,y1)
    plt.xticks(rotation=30)
    for a,b,c in zip(x,y2,y1):
        plt.text(a,b+0.1,'{}'.format(str(c*100)[:4]+'%'),ha = 'center',va = 'bottom')
    plt.xlabel('客户类型')
    plt.ylabel('数量/占比')
    plt.tight_layout()
    plt.savefig('{}.jpg'.format(name))
    plt.show()
    plt.close()
visual_show(rfm_count['客户类型'],rfm_count['人数占比'],rfm_count['客户人数'],'客户人数')
visual_show(rfm_count['客户类型'],rfm_count['金额占比'],rfm_count['消费金额'],'客户消费')
