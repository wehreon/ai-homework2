# ai-homework2
## 1. 任务拆解与 AI 协作策略
在这次任务中，我先让ai明确了库使用的技术工程规范；第二步，我尝试将IDData.csv文件直接喂给ai做数据读取，发现文件过大在一个对话中难以处理，于是我们使用了一个探查文件，
将列名列表，前后5行数据，数据类型做了一个概览，并让ai先对处理的数据做一个预测，到此初步完成了数据读取，接下来开始拆解6项任务。
6项任务包括数据预处理；时间分布分析；线路站点分析；高峰小时系数计算；线路驾驶员信息批量导出；以及服务绩效排名与热力图。

## 2. 核心 Prompt 迭代记录
在任务3：线路站点分析中，ai对seaborn的使用不熟练，其中线路图从0到50000跨度非常大，又没有加标注，非常不清晰；此外底部的均值和标准差重叠在一起，效果类似乱码。
初代 Prompt：...
# -------------------- seaborn水平条形图可视化 --------------------
print("\n【绘制线路平均搭乘站点数条形图】")
top15 = route_stats.head(15).copy()
top15['mean_stops'] = top15['mean_stops'].round(1)
top15['std_stops'] = top15['std_stops'].round(1)
plt.figure(figsize=(14, 9))  # 画布加宽
sns.barplot(
    data=top15,
    x='mean_stops',
    y='线路号',
    palette='Blues_d',
    errorbar=None,
    capsize=0.3,
    label='平均搭乘站点数'
)
for i, (_, row) in enumerate(top15.iterrows()):
    if i == 0:
        plt.errorbar(
            x=row['mean_stops'],
            y=i,
            xerr=row['std_stops'],
            capsize=5,
            fmt='none',
            color='black',
            linewidth=1.5,
            label='±1个标准差（离散程度）'
        )
    else:
        plt.errorbar(
            x=row['mean_stops'],
            y=i,
            xerr=row['std_stops'],
            capsize=5,
            fmt='none',
            color='black',
            linewidth=1.5
        )
for i, (_, row) in enumerate(top15.iterrows()):
    label_text = f"{row['mean_stops']}±{row['std_stops']}"
    plt.text(
        x=row['mean_stops'],
        y=i - 0.2,
        s=label_text,
        va='top',
        ha='center',
        fontsize=9,
        color='black'
    )
plt.xlabel('平均搭乘站点数', fontsize=12)
plt.ylabel('线路号', fontsize=12)
plt.title('各线路乘客平均搭乘站点数（前15条线路）', fontsize=14)
x_max_value = top15['mean_stops'].max() + top15['std_stops'].max()
plt.xlim(0, x_max_value * 1.2)  # 留20%空白
plt.gca().xaxis.set_major_formatter(plt.FormatStrFormatter('%.1f'))
plt.legend(loc='lower right', fontsize=10)
plt.tight_layout()
plt.savefig('route_stops.png', dpi=150, bbox_inches='tight')
print("条形图已保存为 route_stops.png")
plt.show()


我跟ai反馈了以上问题，对每个选出来的线路进行了线路标注，同时把均值+标准差的呈现形式强行调整到两位小数，在不影响数据观察的同时为数据留了空间。
优化后的 Prompt：...
# -------------------- seaborn水平条形图可视化 --------------------
print("\n【绘制线路平均搭乘站点数条形图】")
top15 = route_stats.head(15).copy()
top15['mean_stops'] = top15['mean_stops'].round(2)
top15['std_stops'] = top15['std_stops'].round(2)
top15['线路号'] = top15['线路号'].astype(str)
plt.figure(figsize=(12, 9))
sns.barplot(
    data=top15,
    x='mean_stops',
    y='线路号',
    palette='Blues_d',
    errorbar=None,
    capsize=0.3,
    label='平均搭乘站点数'
)
for i, (_, row) in enumerate(top15.iterrows()):
    if i == 0:
        plt.errorbar(
            x=row['mean_stops'],
            y=i,
            xerr=row['std_stops'],
            capsize=5,
            fmt='none',
            color='black',
            linewidth=1.5,
            label='±1个标准差'
        )
    else:
        plt.errorbar(
            x=row['mean_stops'],
            y=i,
            xerr=row['std_stops'],
            capsize=5,
            fmt='none',
            color='black',
            linewidth=1.5
        )
for i, (_, row) in enumerate(top15.iterrows()):
    label_text = f"线路{row['线路号']}: {row['mean_stops']}±{row['std_stops']}"
    plt.text(
        x=row['mean_stops'] + row['std_stops'] + 0.5,
        y=i,
        s=label_text,
        va='center',
        ha='left',
        fontsize=9,
        color='black'
    )
plt.xlabel('平均搭乘站点数', fontsize=12)
plt.ylabel('线路号', fontsize=12)
plt.title('各线路乘客平均搭乘站点数（前15条线路）', fontsize=14)
x_max = (top15['mean_stops'] + top15['std_stops']).max()
plt.xlim(0, x_max + 8)  # 多留空间给文字
plt.gca().xaxis.set_major_formatter(plt.FormatStrFormatter('%.1f'))
plt.legend(loc='lower right', fontsize=10)
plt.tight_layout()
plt.savefig('route_stops.png', dpi=150, bbox_inches='tight')
print("条形图已保存为 route_stops.png")
plt.show()


## 3. Debug 记录
（记录一次解决报错的过程，例如：时区解析报错 / 热力图中文乱码 / ride_stops=0 导致的结果偏差）
报错现象：在任务6生成热力图时，图表中的中文标题、行列标签显示为方框乱码，如xy轴，标题都错误的显示为方框，这是因为matplotlib/seaborn 默认字体不支持中文显示，需要手动指定系统中存在的中文字体。
解决过程：要想解决就必须在代码前添加中文配置：
import matplotlib.pyplot as plt

plt.rcParams['font.sans-serif'] = ['SimHei', 'Microsoft YaHei', 'DejaVu Sans']#解决中文乱码异常
plt.rcParams['axes.unicode_minus'] = False  # 解决负号显示异常


## 4. 人工代码审查（逐行中文注释）
（贴出任务4 PHF 计算的核心代码，并加上你自己的逐行中文注释）
```python
# 贴入代码及注释
```
