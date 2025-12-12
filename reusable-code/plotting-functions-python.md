# Python 作图函数 Python Plotting Functions

常用的可复用作图函数，主要基于 matplotlib 和 seaborn。

## 基础设置 Basic Setup

### 导入常用库
```python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
import pandas as pd

# 设置中文显示
plt.rcParams['font.sans-serif'] = ['SimHei', 'DejaVu Sans']
plt.rcParams['axes.unicode_minus'] = False

# 设置默认样式
sns.set_style("whitegrid")
plt.rcParams['figure.dpi'] = 100
```

## 通用函数 General Functions

### 保存高质量图片
```python
def save_figure(fig, filename, dpi=300, bbox_inches='tight', 
                transparent=False, formats=['png', 'pdf']):
    """
    保存高质量图片
    
    参数:
        fig: matplotlib figure对象
        filename: 文件名（不含扩展名）
        dpi: 分辨率
        bbox_inches: 边界框设置
        transparent: 是否透明背景
        formats: 保存格式列表
    """
    for fmt in formats:
        fig.savefig(f"{filename}.{fmt}", 
                   dpi=dpi, 
                   bbox_inches=bbox_inches,
                   transparent=transparent)
    print(f"图片已保存: {filename}")
```

### 设置图片风格
```python
def setup_plot_style(style='seaborn-v0_8-darkgrid', figsize=(10, 6), 
                     font_size=12, use_latex=False):
    """
    设置统一的绘图风格
    
    参数:
        style: matplotlib样式
        figsize: 图片大小
        font_size: 字体大小
        use_latex: 是否使用LaTeX渲染
    """
    plt.style.use(style)
    plt.rcParams['figure.figsize'] = figsize
    plt.rcParams['font.size'] = font_size
    
    if use_latex:
        plt.rcParams['text.usetex'] = True
        plt.rcParams['font.family'] = 'serif'
```

## 科学作图函数 Scientific Plotting Functions

### 绘制带误差线的折线图
```python
def plot_line_with_error(x, y_mean, y_std, label=None, 
                         color=None, alpha=0.2, ax=None):
    """
    绘制带误差线（阴影）的折线图
    
    参数:
        x: x轴数据
        y_mean: y轴均值
        y_std: y轴标准差
        label: 图例标签
        color: 线条颜色
        alpha: 阴影透明度
        ax: matplotlib axes对象，如果为None则创建新图
    """
    if ax is None:
        fig, ax = plt.subplots()
    
    line = ax.plot(x, y_mean, label=label, color=color)
    color = line[0].get_color() if color is None else color
    
    ax.fill_between(x, y_mean - y_std, y_mean + y_std, 
                     alpha=alpha, color=color)
    
    return ax
```

### 绘制多子图
```python
def create_subplots(n_plots, n_cols=3, figsize=None, **kwargs):
    """
    创建多子图布局
    
    参数:
        n_plots: 子图总数
        n_cols: 列数
        figsize: 图片大小，如果为None则自动计算
        **kwargs: 传递给plt.subplots的其他参数
    """
    n_rows = int(np.ceil(n_plots / n_cols))
    
    if figsize is None:
        figsize = (5 * n_cols, 4 * n_rows)
    
    fig, axes = plt.subplots(n_rows, n_cols, figsize=figsize, **kwargs)
    axes = np.array(axes).flatten()
    
    # 隐藏多余的子图
    for idx in range(n_plots, len(axes)):
        axes[idx].set_visible(False)
    
    return fig, axes[:n_plots]
```

### 绘制热图
```python
def plot_heatmap(data, annot=True, fmt='.2f', cmap='coolwarm',
                center=None, ax=None, **kwargs):
    """
    绘制热图
    
    参数:
        data: 2D数组或DataFrame
        annot: 是否显示数值
        fmt: 数值格式
        cmap: 颜色映射
        center: 颜色中心值
        ax: matplotlib axes对象
        **kwargs: 传递给seaborn.heatmap的其他参数
    """
    if ax is None:
        fig, ax = plt.subplots(figsize=(10, 8))
    
    sns.heatmap(data, annot=annot, fmt=fmt, cmap=cmap,
               center=center, ax=ax, **kwargs)
    
    return ax
```

### 绘制箱线图
```python
def plot_boxplot(data, x=None, y=None, hue=None, 
                order=None, showmeans=True, ax=None):
    """
    绘制箱线图
    
    参数:
        data: DataFrame或数组
        x, y, hue: 数据列名
        order: x轴顺序
        showmeans: 是否显示均值
        ax: matplotlib axes对象
    """
    if ax is None:
        fig, ax = plt.subplots(figsize=(10, 6))
    
    sns.boxplot(data=data, x=x, y=y, hue=hue, 
               order=order, ax=ax,
               showmeans=showmeans,
               meanprops={"marker": "o", 
                         "markerfacecolor": "white", 
                         "markeredgecolor": "black",
                         "markersize": 8})
    
    return ax
```

### 绘制相关性矩阵
```python
def plot_correlation_matrix(df, method='pearson', 
                           mask_upper=True, annot=True, 
                           figsize=(12, 10)):
    """
    绘制相关性矩阵热图
    
    参数:
        df: DataFrame
        method: 相关性计算方法 ('pearson', 'spearman', 'kendall')
        mask_upper: 是否遮盖上三角
        annot: 是否显示相关系数
        figsize: 图片大小
    """
    # 计算相关性矩阵
    corr = df.corr(method=method)
    
    # 创建遮罩
    mask = None
    if mask_upper:
        mask = np.triu(np.ones_like(corr, dtype=bool))
    
    # 绘图
    fig, ax = plt.subplots(figsize=figsize)
    sns.heatmap(corr, mask=mask, annot=annot, 
               cmap='coolwarm', center=0,
               square=True, linewidths=1,
               cbar_kws={"shrink": 0.8}, ax=ax)
    
    plt.title(f'Correlation Matrix ({method.capitalize()})', 
             fontsize=16, pad=20)
    
    return fig, ax
```

## 自定义配色方案 Custom Color Schemes

```python
# 科研常用配色
SCIENTIFIC_COLORS = {
    'blue': '#1f77b4',
    'orange': '#ff7f0e',
    'green': '#2ca02c',
    'red': '#d62728',
    'purple': '#9467bd',
    'brown': '#8c564b',
    'pink': '#e377c2',
    'gray': '#7f7f7f',
    'olive': '#bcbd22',
    'cyan': '#17becf'
}

# Nature期刊风格配色
NATURE_COLORS = ['#E64B35', '#4DBBD5', '#00A087', '#3C5488', 
                '#F39B7F', '#8491B4', '#91D1C2', '#DC0000']

def get_color_palette(name='scientific', n_colors=None):
    """
    获取配色方案
    
    参数:
        name: 配色方案名称
        n_colors: 需要的颜色数量
    """
    if name == 'scientific':
        colors = list(SCIENTIFIC_COLORS.values())
    elif name == 'nature':
        colors = NATURE_COLORS
    else:
        colors = sns.color_palette(name, n_colors)
    
    if n_colors:
        return colors[:n_colors]
    return colors
```

## 使用示例 Usage Examples

```python
# 示例1: 带误差线的折线图
x = np.linspace(0, 10, 50)
y1_mean = np.sin(x)
y1_std = 0.1 * np.ones_like(x)
y2_mean = np.cos(x)
y2_std = 0.15 * np.ones_like(x)

fig, ax = plt.subplots(figsize=(10, 6))
plot_line_with_error(x, y1_mean, y1_std, label='Sin', ax=ax)
plot_line_with_error(x, y2_mean, y2_std, label='Cos', ax=ax)
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.legend()
ax.set_title('Example: Line Plot with Error Bands')
plt.tight_layout()

# 示例2: 保存图片
save_figure(fig, 'example_plot', dpi=300, formats=['png', 'pdf'])
```

## 注意事项 Notes

1. 使用前确保安装必要的库: `pip install matplotlib seaborn numpy pandas`
2. 中文显示需要系统安装相应字体
3. 保存PDF格式时注意字体嵌入问题
4. 高DPI图片文件较大，根据需要选择合适的分辨率

## 标签 Tags
`python` `matplotlib` `seaborn` `plotting` `visualization` `科研作图`
