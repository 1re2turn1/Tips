# 数据处理工具 Data Processing Tools

常用的数据处理函数和代码片段。

## Pandas 工具函数 Pandas Utilities

### 快速数据概览
```python
import pandas as pd
import numpy as np

def quick_summary(df, show_sample=True, n_sample=5):
    """
    快速查看DataFrame的基本信息
    
    参数:
        df: pandas DataFrame
        show_sample: 是否显示样本数据
        n_sample: 显示的样本数量
    """
    print("=" * 50)
    print("数据形状 (Shape):")
    print(f"  行数: {df.shape[0]}, 列数: {df.shape[1]}")
    print()
    
    print("数据类型 (Data Types):")
    print(df.dtypes)
    print()
    
    print("缺失值统计 (Missing Values):")
    missing = df.isnull().sum()
    missing_pct = 100 * missing / len(df)
    missing_df = pd.DataFrame({
        '缺失数量': missing,
        '缺失比例(%)': missing_pct
    })
    print(missing_df[missing_df['缺失数量'] > 0])
    print()
    
    print("数值列统计 (Numeric Columns Statistics):")
    print(df.describe())
    print()
    
    if show_sample:
        print(f"前 {n_sample} 行数据:")
        print(df.head(n_sample))
    
    print("=" * 50)
```

### 处理缺失值
```python
def handle_missing_values(df, strategy='drop', fill_value=None, 
                         columns=None, threshold=0.5):
    """
    处理缺失值
    
    参数:
        df: pandas DataFrame
        strategy: 处理策略 ('drop', 'mean', 'median', 'mode', 'forward', 'backward', 'constant')
        fill_value: 当strategy='constant'时使用的填充值
        columns: 要处理的列，None表示所有列
        threshold: 当strategy='drop'时，删除缺失比例超过threshold的列
    
    返回:
        处理后的DataFrame
    """
    df_copy = df.copy()
    
    if columns is None:
        columns = df_copy.columns
    
    if strategy == 'drop':
        # 删除缺失值过多的列
        for col in columns:
            if df_copy[col].isnull().sum() / len(df_copy) > threshold:
                df_copy = df_copy.drop(col, axis=1)
        # 删除包含缺失值的行
        df_copy = df_copy.dropna(subset=[c for c in columns if c in df_copy.columns])
    
    elif strategy == 'mean':
        df_copy[columns] = df_copy[columns].fillna(df_copy[columns].mean())
    
    elif strategy == 'median':
        df_copy[columns] = df_copy[columns].fillna(df_copy[columns].median())
    
    elif strategy == 'mode':
        for col in columns:
            df_copy[col].fillna(df_copy[col].mode()[0], inplace=True)
    
    elif strategy == 'forward':
        df_copy[columns] = df_copy[columns].ffill()
    
    elif strategy == 'backward':
        df_copy[columns] = df_copy[columns].bfill()
    
    elif strategy == 'constant':
        df_copy[columns] = df_copy[columns].fillna(fill_value)
    
    return df_copy
```

### 数据标准化和归一化
```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler

def normalize_data(df, method='standard', columns=None):
    """
    数据标准化或归一化
    
    参数:
        df: pandas DataFrame
        method: 'standard' (标准化) 或 'minmax' (归一化)
        columns: 要处理的列，None表示所有数值列
    
    返回:
        处理后的DataFrame和scaler对象
    """
    df_copy = df.copy()
    
    if columns is None:
        columns = df_copy.select_dtypes(include=[np.number]).columns
    
    if method == 'standard':
        scaler = StandardScaler()
    elif method == 'minmax':
        scaler = MinMaxScaler()
    else:
        raise ValueError("method must be 'standard' or 'minmax'")
    
    df_copy[columns] = scaler.fit_transform(df_copy[columns])
    
    return df_copy, scaler
```

## 数据分组和聚合 Grouping and Aggregation

### 智能分组聚合
```python
def smart_groupby(df, group_cols, agg_dict=None, include_count=True):
    """
    智能分组聚合
    
    参数:
        df: pandas DataFrame
        group_cols: 分组列
        agg_dict: 聚合字典，如 {'col1': 'mean', 'col2': ['min', 'max']}
        include_count: 是否包含计数
    
    返回:
        聚合后的DataFrame
    """
    if agg_dict is None:
        # 默认：数值列使用mean，分类列使用first
        agg_dict = {}
        for col in df.columns:
            if col not in group_cols:
                if df[col].dtype in [np.float64, np.int64]:
                    agg_dict[col] = 'mean'
                else:
                    agg_dict[col] = 'first'
    
    result = df.groupby(group_cols).agg(agg_dict)
    
    if include_count:
        result['count'] = df.groupby(group_cols).size()
    
    return result.reset_index()
```

## 时间序列处理 Time Series Processing

### 时间特征提取
```python
def extract_time_features(df, date_column, drop_original=False):
    """
    从日期列提取时间特征
    
    参数:
        df: pandas DataFrame
        date_column: 日期列名
        drop_original: 是否删除原始日期列
    
    返回:
        添加时间特征后的DataFrame
    """
    df_copy = df.copy()
    
    # 确保是datetime类型
    df_copy[date_column] = pd.to_datetime(df_copy[date_column])
    
    # 提取特征
    df_copy[f'{date_column}_year'] = df_copy[date_column].dt.year
    df_copy[f'{date_column}_month'] = df_copy[date_column].dt.month
    df_copy[f'{date_column}_day'] = df_copy[date_column].dt.day
    df_copy[f'{date_column}_dayofweek'] = df_copy[date_column].dt.dayofweek
    df_copy[f'{date_column}_dayofyear'] = df_copy[date_column].dt.dayofyear
    df_copy[f'{date_column}_quarter'] = df_copy[date_column].dt.quarter
    df_copy[f'{date_column}_is_weekend'] = df_copy[date_column].dt.dayofweek.isin([5, 6]).astype(int)
    
    if drop_original:
        df_copy = df_copy.drop(date_column, axis=1)
    
    return df_copy
```

### 移动窗口计算
```python
def rolling_statistics(df, column, windows=[7, 14, 30], 
                      stats=['mean', 'std', 'min', 'max']):
    """
    计算移动窗口统计量
    
    参数:
        df: pandas DataFrame
        column: 要计算的列
        windows: 窗口大小列表
        stats: 统计量列表
    
    返回:
        添加移动窗口统计量的DataFrame
    """
    df_copy = df.copy()
    
    for window in windows:
        for stat in stats:
            col_name = f'{column}_rolling_{window}_{stat}'
            if stat == 'mean':
                df_copy[col_name] = df_copy[column].rolling(window=window).mean()
            elif stat == 'std':
                df_copy[col_name] = df_copy[column].rolling(window=window).std()
            elif stat == 'min':
                df_copy[col_name] = df_copy[column].rolling(window=window).min()
            elif stat == 'max':
                df_copy[col_name] = df_copy[column].rolling(window=window).max()
            elif stat == 'sum':
                df_copy[col_name] = df_copy[column].rolling(window=window).sum()
    
    return df_copy
```

## 数据验证 Data Validation

### 检查数据质量
```python
def check_data_quality(df):
    """
    检查数据质量问题
    
    参数:
        df: pandas DataFrame
    
    返回:
        包含质量问题的字典
    """
    issues = {}
    
    # 检查重复行
    duplicates = df.duplicated().sum()
    if duplicates > 0:
        issues['duplicate_rows'] = duplicates
    
    # 检查缺失值
    missing = df.isnull().sum()
    if missing.sum() > 0:
        issues['missing_values'] = missing[missing > 0].to_dict()
    
    # 检查常量列（所有值相同）
    constant_cols = [col for col in df.columns if df[col].nunique() == 1]
    if constant_cols:
        issues['constant_columns'] = constant_cols
    
    # 检查数值列的异常值（使用IQR方法）
    numeric_cols = df.select_dtypes(include=[np.number]).columns
    outliers = {}
    for col in numeric_cols:
        Q1 = df[col].quantile(0.25)
        Q3 = df[col].quantile(0.75)
        IQR = Q3 - Q1
        outlier_count = ((df[col] < (Q1 - 1.5 * IQR)) | 
                        (df[col] > (Q3 + 1.5 * IQR))).sum()
        if outlier_count > 0:
            outliers[col] = outlier_count
    
    if outliers:
        issues['outliers'] = outliers
    
    return issues
```

## 数据导入导出 Import/Export

### 智能读取CSV
```python
def smart_read_csv(filepath, **kwargs):
    """
    智能读取CSV，自动处理常见问题
    
    参数:
        filepath: 文件路径
        **kwargs: 传递给pd.read_csv的其他参数
    
    返回:
        pandas DataFrame
    """
    # 尝试不同的编码
    encodings = ['utf-8', 'gbk', 'gb2312', 'latin1']
    
    for encoding in encodings:
        try:
            df = pd.read_csv(filepath, encoding=encoding, **kwargs)
            print(f"成功使用 {encoding} 编码读取文件")
            return df
        except UnicodeDecodeError:
            continue
        except Exception as e:
            print(f"读取文件时出错: {e}")
            raise
    
    raise ValueError("无法使用任何编码读取文件")
```

## 使用示例 Usage Examples

```python
# 示例1: 快速查看数据
import pandas as pd
df = pd.read_csv('data.csv')
quick_summary(df)

# 示例2: 处理缺失值
df_clean = handle_missing_values(df, strategy='mean')

# 示例3: 数据标准化
df_normalized, scaler = normalize_data(df, method='standard')

# 示例4: 提取时间特征
df_with_time = extract_time_features(df, 'date_column')

# 示例5: 检查数据质量
quality_issues = check_data_quality(df)
print("数据质量问题:", quality_issues)
```

## 注意事项 Notes

1. 处理大数据集时注意内存使用
2. 使用`.copy()`避免修改原始数据
3. 标准化/归一化后保存scaler用于新数据转换
4. 时间序列数据注意排序

## 标签 Tags
`python` `pandas` `data-processing` `数据处理` `numpy` `sklearn`
