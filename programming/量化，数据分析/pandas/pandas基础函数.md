<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2023-07-03 17:43:34
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2023-07-04 13:16:07
 * @FilePath: \studys\programming\量化，数据分析\pandas\pandas基础函数.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
### 1. 导入模块
```python
import pandas as pd
import numpy as np

```
### 2. 读取数据和保存数据
```python
# 2.1 从CSV文件读取数据,编码'gbk'
pd.read_csv(filename, encoding='gbk')

# 2.2 读取前6行,当数据量比较大时，可以只读取前n行
pd.read_csv(filename, encoding='gbk', nrows = 6)

# 2.3 第一列作为行索引,忽略列索引
pd.read_csv(filename, encoding='gbk', header=None, index_col=0)

# 2.4 读取时忽略第1/3/5行和最后两行
pd.read_csv(filename, encoding='gbk', skiprows=[1,3,5], skipfooter=2, engine='python')

# 2.5 从限定分隔符（','）的文件或文本读取数据
pd.read_table(filename, sep=',', encoding='gbk')

# 2.6 保存数据
# 保存为csv文件
df.to_csv('test_ison.csv')
# 保存为xlsx文件
df.to_excel('test_xlsx.xlsx',index=False)
# 保存为ison文件
df.to_json('test_json.txt')

```
### 查看数据信息
```python
# 3.1 查看前n行
df.head(3)

# 3.2 查看后n行
df.tail(3)

# 3.3 查看行数和列数
df.shape

# 3.4查看列索引
df.columns

# 3.5 查看行索引
df.index

# 3.6 查看索引、数据类型和内存信息
df.info()

# 3.7 查看数值型列的汇总统计
df.describe()

# 3.8 查看每一列的唯一值和计数
df.apply(pd.Series.value_counts)

```
### 4. 数据处理
```python
# 4.1 重命名列名
df.columns = ['姓名','性别','语文','数学','英语','城市','省份']

# 4.2 选择性更改列名
df.rename(columns={'姓名': '姓--名','性别': '性--别'},inplace=True)

# 4.3 批量更改索引
df.rename(lambda x: x + 10)

# 4.4 批量更改列名
df.rename(columns=lambda x: x + '_1')

# 4.5 设置姓名列为行索引
df.set_index('姓名')

# 4.6 检查哪些列包含缺失值
df.isnull().any()

# 4.7 统计各列空值
df.isnull().sum()

# 4.8 删除本列中空值的行
df[df['数学'].notnull()]
df[~df['数学'].isnull()]

# 4.9 仅保留本列中是空值的行
df[df['数学'].isnull()]
df[~df['数学'].notnull()]

# 4.10 去掉某行
df.drop(0, axis=0)

# 4.11 去掉某列
df.drop('英语', axis=1)

# 4.12 删除所有包含空值的行
df.dropna()

# 4.13 删除行里全都是空值的行
df.dropna(how = 'all')

# 4.14 删除所有包含空值的列
df.dropna(axis=1)

# 4.15 保留至少有6个非空值的行
df.dropna(thresh=6)

# 4.16 保留至少有11个非空值的列
df.dropna(axis=1,thresh=11)

# 4.17 行数据向下填充
df.fillna(method = 'ffill')

# 4.18 列数据向右填充
df.fillna(method = 'ffill',axis=1)

# 4.19 用0替换所有的空值
df.fillna(0)

# 4.20 强制转换数据类型
df_t1 = df.dropna()
df_t1['语文'].astype('int')

# 4.21 查看有多少不同的城市
df['城市'].unique()

# 4.22 单值替换
df.replace('苏州', '南京')

# 4.23 多值替换
df.replace({'苏州':'南京','广州':'深圳'})
df.replace(['苏州','广州'],['南京','深圳'])

# 4.24 多值替换单值
df.replace(['深圳','广州'],'东莞')

# 4.25 替换某列，显示需要加inplace=True
df['城市'] = df['城市'].replace('苏州', '南京')

# 4.26 拆分某列,生成新的Dataframe
#sep表示用于分割的字符；
#n表格分割成多少列；
#expand表示是否展开为数据款，True输出Series，False输出Dataframe。
#series，只是一个一维数据结构，它由index和value组成。
#dataframe，是一个二维结构，除了拥有index和value之外，还拥有column。
#联系：
#dataframe由多个series组成，无论是行还是列，单独拆分出来都是一个series。
df1 = df['姓名'].str.split('-',expand=True)
df1.columns = ['学号','姓名']

# 4.27 某一列类型转换，注意该列类型要一致，包括（NaN）
df1.dropna(inplace = True)
df1['语文'] = df1['语文'].apply(int)

```
### 5. 数据切片、筛选
```python
# 5.1 输出城市为上海
df[df['城市']=='上海']

# 5.2 输出城市为上海或广州
df[df['城市'].isin(['上海','广州'])]

# 5.3 输出城市名称中含有‘海’字的行
df[df['城市'].str.contains("海", na=False)]

# 5.4 输出城市名称以‘海’字开头的行
df[df['城市'].str.startswith("海", na=False)]

# 5.5 输出城市名称以‘海’字结尾的行
df[df['城市'].str.endswith("海", na=False)]

# 5.6 输出所有姓名，缺失值用Null填充
df['姓名'].str.cat(sep='、',na_rep='Null')

# 5.7 重置索引
df2 = df1.set_index('学号')

# 5.8 前两行
df2[:2]

# 5.9 后两行
df2[-2:]

# 5.10 2-8行
df2[2:8]

# 5.11 每隔3行读取
df2[::3]

# 5.12 2-8行,步长为2，即第2/4/6行
df2[2:8:2]

# 5.13 选取'语文','数学','英语'列
df2[['语文','数学','英语']]

# df.loc[]只能使用标签索引，不能使用整数索引，通过便签索引切边进行筛选时，前闭后闭
# 5.14 学号为'001'的行，所有列
df2.loc['001', :]

# 5.15 学号为'001'或'003'的行，所有列
df2.loc[['001','003'], :]

# 5.16 学号为'001'至'009'的行，所有列
df2.loc['001':'009', :]

# 5.17 列索引为'姓名'，所有行
df2.loc[:, '姓名']

# 5.18 列索引为'姓名'至‘城市’，所有行
df2.loc[:, '姓名':'城市']

# 5.19 语文成绩大于80的行
df2.loc[df2['语文']>80,:]
df2.loc[df2.loc[:,'语文']>80, :]
df2.loc[lambda df2:df2['语文'] > 80, :]

# 5.20 语文成绩大于80的人的学号和姓名
df2.loc[df2['语文']>80,['姓名','城市']]

# 5.21 输出'赵四'和'周七'的各科成绩
df2.loc[df2['姓名'].isin(['赵四','周七']),['姓名','语文','数学','英语']]

# # df.iloc[]只能使用整数索引，不能使用标签索引，通过整数索引切边进行筛选时，前闭后开
# 5.22 选取第2行
df2.iloc[1, :]

# 5.23 选取前3行
df2.iloc[:3, :]

# 5.24 选取第2行、第4行、第6行
df2.iloc[[1,3,5],:]

# 5.25 选取第2列
df2.iloc[:, 1]

# 5.26 选取前3列
df2.iloc[:, 0:3]

# 5.27 选取第3行的第3列
df2.iloc[3, 3]

# 5.28 选取第1列、第3列和第4列
df2.iloc[:, [0,2,3]]

# 5.29 选取第2行的第1列、第3列、第4列
df2.iloc[1, [0,2,3]]

# 5.30 选取前3行的前3列
df2.iloc[:3, :3]

```
### 6. 数据排序
```python
# 6.1 重置索引
df_last = df1.reset_index(drop=True)

# 6.2 按照语文成绩升序排序，默认升序排列
df_last.sort_values('语文')

# 6.3 按照数学成绩降序排序
df_last.sort_values('数学', ascending=False)

# 6.4 先按语文成绩升序排列，再按数学成绩降序排列
df_last.sort_values(['语文','数学'], ascending=[True,False])

# 6.5 语文成绩80及以上
df_last[df_last['语文']>=80]
df_last.query('语文 > 80')

# 6.6 语文成绩80及以上以及数学成绩90分及以上
df_last[(df_last['语文']>=80) & (df_last['数学']>=90)]

# 6.7 语文成绩80及以上或数学成绩90分及以上
df_last[(df_last['语文']>=80) | (df_last['数学']>=90)]

# 6.8 输出成绩100的行和列号
row, col = np.where(df_last.values == 100)

# 6.9 增加一列“省份-城市”
df_last['省份-城市'] = df_last['省份'] + '-' + df_last['城市']

# 6.10 增加一列总分
df_last['总分'] = df_last[['语文','数学','英语']].sum(axis = 1)

# 6.11 按照总分、语文、数学、英语成绩依次排序
df_last.sort_values(by =['总分','语文','数学','英语'],ascending=False )

# 6.12 新增一列表示学生语文成绩等级的列（优秀、良好、中等、不及格）
def get_letter_grade(score):
    '''
    定义一个函数，根据分数返回相应的等级
    '''
    if score>=90:
        return '优秀'
    elif score>=80:
        return '良好'
    elif score>=60:
        return '中等'
    else:
        return '不及格'
    
df_last['语文等级'] = df_last['语文'].apply(lambda score: get_letter_grade(score))

```
### 7. 数据分组
```python
# 7.1 一列分组
df2.groupby('省份').groups

# 7.2 多列分组
df2.groupby(['省份','城市']).groups

# 7.3 每组的统计数据（横向显示）
df2.groupby('省份').describe()

# 7.4 每组的统计数据（纵向显示）
df2.groupby('省份').describe().unstack()

# 7.5 查看指定列的统计信息
df2.groupby('省份').describe()['语文']

# 7.6 分组大小
df2.groupby('省份').count()
df2.groupby('省份').agg(np.size)

# 7.7 分组成绩最大值
df2.groupby('省份').max()
df2.groupby('省份').agg(np.max)

# 7.8 分组成绩最小值
df2.groupby('省份').min()
df2.groupby('省份').agg(np.min)

# 7.9 分组成绩总和
df2.groupby('省份').sum()
df2.groupby('省份').agg(np.sum)

# 7.10 分组平均成绩
df2.groupby('省份').mean()
df2.groupby('省份').agg(np.mean)

# 7.11 按省份分组，计算英语成绩总分和平均分
df2.groupby('省份')['英语'].agg([np.sum, np.mean])

# 7.12 按省份、城市分组计算平均成绩
df2.groupby(['省份','城市']).agg(np.mean)

# 7.13 不同列不同的计算方法
df2.groupby('省份').agg({'语文': sum, # 总和
                        '数学': 'count', # 总数
                        '英语':'mean'}) # 平均

# 7.14 性别分别替换为1/0
df2 = df2.dropna()
df2['性别'] = df2['性别'].map({'男':1, '女':0})

# 7.15 增加一列按省份分组的语文平均分
df2['语文平均分'] = df2.groupby('省份')['语文'].transform('mean')

# 7.16 输出语文成绩最高的男生和女生（groupby默认会去掉空值）
def get_max(g):
    df = g.sort_values('语文',ascending=True)
    print(df)
    return df.iloc[-1,:]

df2.groupby('性别').apply(get_max)

# 7.17 按列省份、城市进行分组，计算语文、数学、英语成绩最大值的透视表
df.pivot_table(index=['省份','城市'], values=['语文','数学','英语'], aggfunc=max)

```
### 8. 数据统计
```python
# 8.1 数据汇总统计
df.describe()

# 8.2 列中非空值的个数
df.count()

# 8.3 列最小值
df.min()

# 8.4 列最大值
df.max()

# 8.5 列均值
df.mean()

# 8.6 列中位数
df.median()

# 8.7 列与列之间的相关系数
df.corr()

# 8.8 列的标准差
df.std()

# 8.9 语文成绩指标
# 对语文列求和
sum0 = df_last['语文'].sum()
# 语文成绩方差
var = df_last['语文'].var()
# 语文成绩标准差
std = df_last['语文'].std()
# 语文平均分
mean = df_last['语文'].mean()

print('语文总分:',sum0)
print('语文平均分:',mean)
print('语文成绩标准差:',std)
print('语文成绩方差:',var)

# 8.10 三个科目的指标
mean = df_last[['语文','数学','英语']].mean()
var  = df_last[['语文','数学','英语']].var()
total = df_last[['语文','数学','英语']].sum()
std = df_last[['语文','数学','英语']].std()
rows = [total,mean,var,std]
# 索引列表
index = ['总分','平均分','方差','标准差']
# 根据指定索引和行构造 DataFrame 对象
df_tmp = pd.DataFrame(rows,index=index)

```
### 9. 表格样式
```python
# 9.1 示例数据
df = pd.read_csv(filename, encoding='gbk')

# 9.2 列重命名
df.columns = ['姓名','性别','语文','数学','英语','城市','省份']

# 9.3 设置空值背景红色
df.style.highlight_null(null_color = 'red')

# 9.4 最大数据高亮
df.style.highlight_max()

# 9.5 最小数据高亮
df.style.highlight_min()

# 9.6 部分列最大数据高亮
df.style.apply(highlight_max, subset=['语文', '数学'])

# 9.7 部分列数据高亮（Dataframe全为数据）
df3 = df[['语文','数学','英语']]
def highlight_max(s):
    is_max = s == s.max()
    return ['background-color: yellow' if v else '' for v in is_max]

df3.style.apply(highlight_max)

# 9.8 95分以上显示红色
def color_negative_red(val):
    color = 'red' if val > 95.0 else 'black'
    return 'color: %s' % color

df3.style.applymap(color_negative_red)

# 9.9 混合
df3.style.applymap(color_negative_red).apply(highlight_max)

# 9.10 设置float类型列数据大于80.0的背景高亮
yellow_css = 'background-color: yellow'
sfun = lambda x: yellow_css if type(x) == float and x > 80.0 else ''
df3.style.applymap(sfun)

# 9.11 设置数学成绩大于80.0分的行背景高亮
yellow_css = 'background-color: yellow'
sfun = lambda x: [yellow_css]*len(x) if x.数学 > 80.0 else ['']*len(x)
df3.style.apply(sfun, axis=1)

# 9.12 设置数学成绩大于95.0的行数据颜色为红色
def row_color(s):
    if s.数学 > 95:
        return ['color: red']*len(s)
    else:
        return ['']*len(s)

df3.style.apply(row_color, axis=1)

# 9.13 显示热度图
import seaborn as sns
cm = sns.light_palette("green", as_cmap=True)
df3.style.background_gradient(cmap=cm)

```


