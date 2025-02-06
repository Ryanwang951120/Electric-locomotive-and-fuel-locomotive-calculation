import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.font_manager as fm


# 設定中文字體 (解決亂碼問題)
plt.rcParams['font.family'] = 'SimHei'  # 使用黑體
plt.rcParams['axes.unicode_minus'] = False  # 避免負號顯示錯誤

# 定義變數範圍
K_values = np.arange(100, 5000, 100)  # 年行駛里程從100到2400

# 燃油機車參數
fuel_cost_per_km = 0.8  # 油資成本 (元/km)
maintenance_cost_per_km_fuel = 2.14  # 維護費用 (元/km)
other_cost_per_km_fuel = 0.85  # 其他固定成本 (元/km)

# 電動機車參數
electricity_cost_per_km = 0.15  # 電費成本 (元/km)
maintenance_cost_per_km_electric = 1.28  # 維護費用 (元/km)
other_cost_per_km_electric = 0.51  # 其他固定成本 (元/km)
subscription_cost = 5988  # 換電吃到飽 (固定年費)

# 計算各成本
fuel_total_costs = (fuel_cost_per_km + maintenance_cost_per_km_fuel + other_cost_per_km_fuel) * K_values
electric_total_costs = (electricity_cost_per_km + maintenance_cost_per_km_electric + other_cost_per_km_electric) * K_values + subscription_cost

# 找出交點
cost_diff = fuel_total_costs - electric_total_costs
intersection_idx = np.where(np.diff(np.signbit(cost_diff)))[0]
if len(intersection_idx) > 0:
    # 使用線性插值找出更精確的交點
    idx = intersection_idx[0]
    x1, x2 = K_values[idx], K_values[idx + 1]
    y1, y2 = fuel_total_costs[idx], fuel_total_costs[idx + 1]
    y3, y4 = electric_total_costs[idx], electric_total_costs[idx + 1]
    
    # 線性插值計算交點
    x_intersect = x1 + (x2 - x1) * (y3 - y1) / ((y2 - y1) - (y4 - y3))
    y_intersect = y1 + (y2 - y1) * (x_intersect - x1) / (x2 - x1)

# 建立表格
df = pd.DataFrame({
    "年行駛里程 (K)": K_values,
    "燃油機車總成本 (元)": fuel_total_costs,
    "電動機車總成本 (元)": electric_total_costs
})

# 繪製折線圖
plt.figure(figsize=(10, 5))
plt.plot(df["年行駛里程 (K)"], df["燃油機車總成本 (元)"], label="燃油機車總成本", color="red", marker="o")
plt.plot(df["年行駛里程 (K)"], df["電動機車總成本 (元)"], label="電動機車總成本", color="blue", marker="s")

# 標示交點
if len(intersection_idx) > 0:
    plt.plot(x_intersect, y_intersect, 'ko', markersize=10)  # 加大交點標記
    plt.annotate(f'交點: ({int(x_intersect)}公里, {int(y_intersect)}元)',
                xy=(x_intersect, y_intersect),
                xytext=(x_intersect+100, y_intersect+1000),
                bbox=dict(facecolor='white', edgecolor='black', alpha=0.7),
                arrowprops=dict(arrowstyle='->'))

# 設定標題和標籤
plt.title("燃油機車與電動機車總成本比較", fontsize=14)
plt.xlabel("年行駛里程 (km)", fontsize=12)
plt.ylabel("總成本 (元)", fontsize=12, rotation=90)  # 縱軸標籤垂直排列
plt.yticks(rotation=90)  # 縱軸數值垂直排列
plt.legend()
plt.grid(True)

# 顯示圖表
plt.show()
