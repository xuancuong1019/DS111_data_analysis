# HƯỚNG DẪN A–Z: 5 DASHBOARD TRÊN BỘ DỮ LIỆU STUDENT PERFORMANCE

> Công cụ: **pandas – numpy – matplotlib**. Code ngắn, đúng kiểu slide (`df.plot(kind=...)`, `plt.subplots`, `imshow`).
> Dữ liệu: `student-mat.csv` (395 dòng) + `student-por.csv` (649 dòng), phân tách bằng `;`.
> Môi trường: Jupyter Notebook (VS Code / Colab đều chạy được). Cần matplotlib ≥ 3.5 và pandas ≥ 1.5.

---

## MỤC LỤC

0. Nguyên tắc làm việc (dùng AI nhưng phải hiểu mình đang làm gì)
1. Hiểu dữ liệu: bảng kiểu biến + Data Health Check
2. Khung "Kịch bản phân tích" dùng chung + bộ code nền (helper)
3. Dashboard 1 – Academic Performance
4. Dashboard 2 – Student Risk Monitoring
5. Dashboard 3 – Attendance & Study Habits
6. Dashboard 4 – Educational Support
7. Dashboard 5 – Educational Equity
8. Đối chiếu bài tập trên slide + checklist trước khi nộp
9. Bộ câu hỏi thầy có thể hỏi khi bảo vệ

---

## 0. NGUYÊN TẮC LÀM VIỆC

### 0.1. Chuỗi suy luận bắt buộc cho MỖI biểu đồ

```text
Câu hỏi nghiệp vụ → Cấu trúc dữ liệu (kiểu biến) → Loại biểu đồ + lý do → Code → Insight → Hành động
```

Một biểu đồ không đi qua đủ 6 bước này là "prompt ra để đó", đúng thứ thầy không chấp nhận.

### 0.2. "Phiếu 1 phút" trước khi gõ code (tự điền, ghi vào notebook dạng Markdown cell)

| Câu hỏi tự kiểm | Ví dụ trả lời |
|---|---|
| 1. Biểu đồ này trả lời câu hỏi nghiệp vụ nào? | "Điểm trung bình đổi thế nào từ G1 đến G3?" |
| 2. Biến nào tham gia, thuộc kiểu gì? | G1, G2, G3: định lượng; `subject`: định danh |
| 3. Vì sao chọn loại biểu đồ này, không chọn loại khác? | Có 3 mốc thời gian có thứ tự → line chart; pie không thể hiện xu hướng |
| 4. Kết quả có thể khiến tôi kết luận sai ở đâu? | Dữ liệu gộp 2 môn, thang điểm giống nhau nhưng độ khó có thể khác |

### 0.3. Cách dùng AI mà vẫn "sạch" với thầy

- Dùng AI để **giải thích** vì sao chọn biểu đồ, **gỡ lỗi**, **kiểm tra lập luận**. Không dùng để sinh cả dashboard rồi dán.
- Mọi dòng code trong bài phải giải thích được bằng lời: *"dòng này nhận bảng nào, trả ra bảng nào, hình dạng ra sao"*.
- Khi gặp dòng code không hiểu: tách nhỏ, chạy từng bước, in kết quả trung gian (`print(...)`, `.head()`).
- Số liệu "mục tiêu" của Key Result là **giả định do bạn đặt**. Phải ghi rõ lý do đặt.

### 0.4. Triết lý DA-first (để vừa đạt điểm môn học vừa dùng được cho Portfolio)

| Thầy chấm | Nhà tuyển dụng nhìn |
|---|---|
| Đúng lý thuyết biểu đồ, code ngắn | Có câu hỏi nghiệp vụ rõ ràng |
| Có Goal – Metric – Key Result | Có insight và hành động đề xuất |
| Hiểu cấu trúc dữ liệu | Biết giới hạn của dữ liệu và nói thẳng ra |

---

## 1. HIỂU DỮ LIỆU

### 1.1. Bảng kiểu biến (nền tảng để chọn biểu đồ)

| Nhóm | Biến | Giá trị | Kiểu |
|---|---|---|---|
| Định danh (nominal) | `school` | GP, MS | Phân loại |
| | `sex` | F, M | Phân loại |
| | `address` | U (đô thị), R (nông thôn) | Phân loại |
| | `famsize`, `Pstatus` | LE3/GT3, T/A | Phân loại |
| | `Mjob`, `Fjob`, `reason`, `guardian` | nhiều nhãn | Phân loại |
| | `schoolsup`, `famsup`, `paid`, `activities`, `nursery`, `higher`, `internet`, `romantic` | yes / no | Nhị phân |
| Thứ tự (ordinal) | `Medu`, `Fedu` | 0–4 (0 = không học, 4 = đại học) | Mã hoá có thứ tự |
| | `studytime` | 1–4 (<2h, 2–5h, 5–10h, >10h mỗi tuần) | Mã hoá có thứ tự |
| | `traveltime`, `famrel`, `freetime`, `goout`, `Dalc`, `Walc`, `health` | 1–5 (hoặc 1–4) | Mã hoá có thứ tự |
| Đếm (discrete) | `failures` | số lần trượt môn trước đó (0–3) | Số đếm |
| | `absences` | số buổi vắng (0 đến vài chục, lệch phải) | Số đếm |
| | `age` | 15–22 | Số nguyên |
| Định lượng (quantitative) | `G1`, `G2`, `G3` | điểm 0–20 | Định lượng |

> **Cảnh báo kiểu biến:** `studytime` là **mã 1–4**, không phải số giờ. Không nên tính "giờ học trung bình" từ nó. Hãy xem **phân bố** từng mức.

### 1.2. Giới hạn dữ liệu (phải nói thẳng trong báo cáo)

| Giới hạn | Hệ quả |
|---|---|
| Không có chuyên cần theo ngày hoặc theo tuần | Câu "thời gian học theo ngày trong tuần" trên slide **không làm được**. Dashboard 3 dùng `absences` và `studytime` làm biến đại diện |
| Một số học sinh học cả hai môn (UCI ghi khoảng 382 người trùng ở hai file) | Sau khi nối, **1 dòng = 1 học sinh trong 1 môn**, không phải 1 học sinh. KPI phải ghi "lượt học sinh–môn" |
| Có nhiều học sinh `G3 = 0` | Thường là bỏ thi hoặc bỏ học. Làm kéo trung bình xuống và tạo cột đột biến ở 0 trên histogram |
| Dữ liệu cắt ngang (một thời điểm) | Chỉ nói "có liên quan", không nói "gây ra" |

### 1.3. Nạp dữ liệu và Data Health Check

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

mat = pd.read_csv("student-mat.csv", sep=";")
por = pd.read_csv("student-por.csv", sep=";")
mat["subject"], por["subject"] = "Toán", "Bồ Đào Nha"
df = pd.concat([mat, por], ignore_index=True)   # 1 dòng = 1 học sinh trong 1 môn
```

```python
print(df.shape)                                  # kỳ vọng (1044, 34)
df.info()                                        # kiểu dữ liệu từng cột
print("Thiếu:", df.isna().sum().sum())           # kỳ vọng 0
print("Trùng:", df.duplicated().sum())
print(df[["G1", "G2", "G3", "absences"]].describe())
print("G3 = 0:", (df["G3"] == 0).sum())
```

**Bảng ghi kết quả (điền sau khi chạy, đưa vào báo cáo):**

| Kiểm tra | Kết quả | Kết luận xử lý |
|---|---|---|
| Số dòng, số cột | … | |
| Giá trị thiếu | … | Không cần điền khuyết nếu = 0 |
| Dòng trùng | … | Hai môn của cùng học sinh KHÔNG phải trùng, vì `subject` và điểm khác nhau |
| `G3 = 0` | … học sinh | Giữ lại, nêu rõ là có thể bỏ thi |
| `absences` lệch phải | median … vs mean … | Dùng median hoặc gộp nhóm khi mô tả |

---

## 2. KHUNG "KỊCH BẢN PHÂN TÍCH" VÀ BỘ CODE NỀN

### 2.1. Bốn tiêu chí chọn phương án phân tích (đúng yêu cầu thầy)

| Tiêu chí | Câu hỏi tự hỏi | Ảnh hưởng đến lựa chọn |
|---|---|---|
| 1. Nhu cầu nghiệp vụ | Ai dùng? Họ cần ra quyết định gì? | Chọn KPI nào đứng đầu |
| 2. Nguồn dữ liệu sẵn có | Có biến nào đo đúng thứ cần đo không? | Quyết định dùng biến đại diện hay bỏ câu hỏi |
| 3. Năng lực tổ chức | Trường có người biết code không? Cập nhật bao lâu một lần? | Dashboard tĩnh theo kỳ, đơn giản, đọc nhanh |
| 4. Kiểu biến | Biến là phân loại, thứ tự hay định lượng? | Quyết định loại biểu đồ |

### 2.2. Bảng chọn biểu đồ theo cấu trúc dữ liệu

| Câu hỏi | Kiểu biến | Biểu đồ | Code theo slide |
|---|---|---|---|
| Thay đổi qua các mốc có thứ tự | Định lượng theo mốc | Line chart | `plot(kind="line", marker="o")` |
| So sánh giá trị giữa các nhóm | Phân loại → số | Bar / Column (ghép) | `plot(kind="bar")` |
| Phân bố một biến liên tục | Định lượng | Histogram | `plot(kind="hist", bins=…)` |
| So sánh phân bố giữa các nhóm | Định lượng theo nhóm | Box plot | `df.boxplot(column=…, by=…)` |
| Quan hệ giữa hai biến | Định lượng × định lượng | Scatter | `df.plot.scatter(x=, y=)` |
| Giá trị trung bình theo hai biến phân loại | Phân loại × phân loại → số | Heatmap ma trận | `imshow` + `text` |
| Tiến độ so với mục tiêu | Số so với ngưỡng | Thanh ngang (barh) | `barh` + `axvline` |

Không dùng pie chart cho bài này, vì không thể hiện xu hướng, khó so sánh các nhóm gần nhau và slide không dùng.

### 2.3. Biến dẫn xuất dùng chung (định nghĩa MỘT lần, ghi vào chân trang dashboard)

```python
df["passed"]   = df["G3"] >= 10                       # Đạt (thang 0–20)
df["fail"]     = df["G3"] < 10
df["pass_pct"] = df["passed"] * 100                   # để tính tỷ lệ % bằng mean()
df["support_any"] = df[["schoolsup", "famsup", "paid"]].eq("yes").any(axis=1)

# Nhóm nguy cơ: chỉ dùng tín hiệu SỚM (G2, failures, absences), KHÔNG dùng G3
p75   = df["absences"].quantile(0.75)                 # "vắng nhiều" = top 25%
score = (df["G2"] < 10).astype(int) + (df["failures"] >= 1) + (df["absences"] > p75)
df["risk"] = pd.cut(score, [-1, 0, 1, 3], labels=["Thấp", "Trung bình", "Cao"])
```

**Vì sao định nghĩa nguy cơ như vậy (câu thầy chắc chắn hỏi):**

| Quyết định | Lý do |
|---|---|
| Không dùng G3 để định nghĩa nguy cơ | G3 là kết quả cuối. Nếu dùng nó để xếp nhóm rồi lại so điểm G3 thì là lập luận vòng tròn. Nguy cơ phải xác định **trước** kết quả |
| Dùng G2, `failures`, `absences` | Là 3 tín hiệu có sẵn trước kỳ cuối, giáo viên quan sát được |
| Ngưỡng G2 < 10 | Cùng mức "đạt" của G3 |
| Ngưỡng vắng = percentile 75 | Lấy từ chính dữ liệu, tránh một con số tuỳ tiện |
| 0 điểm = Thấp, 1 = Trung bình, ≥2 = Cao | Quy tắc đếm tín hiệu, dễ giải thích cho giáo viên, không cần mô hình |

### 2.4. Bộ hàm vẽ nền (helper, chạy một lần ở đầu notebook)

Chỉ làm 4 việc: vẽ header, vẽ thẻ KPI, đánh tiêu đề biểu đồ, vẽ thanh tiến độ KR (và heatmap). Tất cả đều là `matplotlib` thuần.

```python
BLUE, GOLD, NAVY = "#4C8BC9", "#E0A93B", "#3B4560"

def canvas(title, goal):
    fig, axs = plt.subplots(2, 2, figsize=(16, 10))
    fig.subplots_adjust(top=0.62, bottom=0.08, left=0.06, right=0.97, hspace=0.6, wspace=0.25)
    fig.add_artist(plt.Rectangle((0.02, 0.86), 0.96, 0.12, transform=fig.transFigure, color=NAVY))
    fig.text(0.04, 0.945, "EDUCATION ANALYTICS / MATPLOTLIB", color="white", fontsize=8)
    fig.text(0.04, 0.905, title, color="white", fontsize=20, weight="bold")
    fig.text(0.04, 0.875, "Goal: " + goal, color="#CBD2E0", fontsize=10)
    return fig, axs.ravel()

def kpi(fig, i, tag, label, value, note):           # thẻ KPI / KR (i = 0..3)
    x = 0.04 + i * 0.24
    fig.text(x, 0.80, f"{tag} · {label}", fontsize=9, color="gray")
    fig.text(x, 0.74, value, fontsize=26, weight="bold", color=BLUE)
    fig.text(x, 0.715, note, fontsize=7, color="gray")

def ttl(ax, n, name, sub):                          # tiêu đề: số thứ tự + loại biểu đồ + câu hỏi
    ax.set_title(f"{n} · {name}\n{sub}", loc="left", fontsize=10)

def kr_bars(ax, names, actual, target):             # tiến độ Key Result (cao hơn là tốt hơn)
    ratio = np.minimum(np.array(actual) / np.array(target), 1)
    ax.barh(names, ratio, color=BLUE)
    ax.axvline(1, color="k", ls="--", lw=1)
    ax.set_xlim(0, 1.4); ax.invert_yaxis()
    ax.set_xticks([0, 0.5, 1], ["0%", "50%", "Mục tiêu"])
    for i, (a, t) in enumerate(zip(actual, target)):
        ax.text(ratio[i] + 0.02, i, f"{a:.1f} / {t:.1f}", va="center")

def heat(ax, pt, cmap="Blues"):                     # heatmap ma trận như slide 71
    im = ax.imshow(pt.values, cmap=cmap, aspect="auto")
    ax.set_xticks(range(pt.shape[1]), [str(c) for c in pt.columns])
    ax.set_yticks(range(pt.shape[0]), [str(r) for r in pt.index])
    for i in range(pt.shape[0]):
        for j in range(pt.shape[1]):
            if not np.isnan(pt.values[i, j]):
                ax.text(j, i, f"{pt.values[i, j]:.1f}", ha="center", va="center")
    plt.colorbar(im, ax=ax)
```

### 2.5. Bố cục chung của cả 5 dashboard (giống bài mẫu của thầy)

```text
┌──────────────────────────────────────────────┐
│ HEADER: tên dashboard + Goal                 │
├──────────┬──────────┬──────────┬─────────────┤
│ KPI      │ KR1      │ KR2      │ KR3         │  ← Metric & Key Result
├──────────┴────┬─────┴──────────┴─────────────┤
│ 01 biểu đồ    │ 02 biểu đồ                   │
├───────────────┼──────────────────────────────┤
│ 03 biểu đồ    │ 04 Theo dõi Key Results      │
└───────────────┴──────────────────────────────┘
 Chân trang: định nghĩa chỉ số
```

### 2.6. Thứ tự cell trong notebook

| Cell | Nội dung |
|---|---|
| 1 | Import + nạp dữ liệu (mục 1.3) |
| 2 | Data Health Check (mục 1.3) |
| 3 | Biến dẫn xuất (mục 2.3) |
| 4 | Helper (mục 2.4) |
| 5–9 | Mỗi cell một dashboard (mục 3 đến 7) |

---

## 3. DASHBOARD 1 – ACADEMIC PERFORMANCE (KẾT QUẢ HỌC TẬP)

### 3.1. Bối cảnh nghiệp vụ

Ban giám hiệu / phòng đào tạo hai trường GP và MS cần biết kết quả học tập chung đang ở mức nào, có cải thiện trong năm không, và nhóm nào đang tụt lại, để quyết định có tổ chức ôn tập hay can thiệp ở môn nào.

### 3.2. Goal – Metric – Key Result

| Thành phần | Nội dung |
|---|---|
| **Goal** | Nâng chất lượng học tập ở hai trường GP và MS |
| **KPI (Metric)** | Điểm trung bình G3 |
| **KR1** | Tỷ lệ đạt (G3 ≥ 10) ≥ 85% |
| **KR2** | Tỷ lệ không tụt điểm (G3 ≥ G1) ≥ 70% |
| **KR3** | Tỷ lệ điểm khá (G3 ≥ 14) ≥ 30% |

> Mục tiêu là **giả định**. Cách đặt: chạy dashboard một lần để xem mức hiện tại (baseline), rồi đặt mục tiêu cao hơn baseline vài điểm phần trăm và ghi rõ lý do.

### 3.3. Kịch bản phân tích

| Tiêu chí | Quyết định |
|---|---|
| Nhu cầu nghiệp vụ | Cần tổng quan nhanh và so sánh theo trường, môn |
| Dữ liệu sẵn có | G1, G2, G3 đo kết quả theo ba mốc, đủ cho xu hướng |
| Năng lực tổ chức | Báo cáo tĩnh theo kỳ, không cần tương tác |
| Giới hạn | Hai môn khác độ khó; không so điểm tuyệt đối giữa môn như so "năng lực" |

### 3.4. Bảng chọn biểu đồ

| # | Câu hỏi | Biến và kiểu | Biểu đồ | Vì sao |
|---|---|---|---|---|
| 01 | Điểm TB đổi thế nào từ G1 đến G3, mỗi môn? | G1–G3 (định lượng, có thứ tự mốc), `subject` | Line | Đúng mục đích line: xu hướng qua mốc có thứ tự, so nhiều chuỗi |
| 02 | Phân bố G3 có dạng gì? Bao nhiêu em dưới ngưỡng đạt? | G3 (định lượng liên tục) | Histogram + đường ngưỡng 10 | Histogram cho thấy hình dạng phân bố |
| 03 | Phân bố G3 khác nhau thế nào theo môn và trường? | G3 theo `subject` × `school` | Box plot | So sánh phân bố giữa các nhóm bằng 5 số thống kê |
| 04 | KR đạt đến đâu so với mục tiêu? | 3 tỷ lệ % và ngưỡng | Thanh ngang | Thể hiện tiến độ |

### 3.5. Code

```python
fig, ax = canvas("Academic Performance Dashboard", "Nâng chất lượng học tập ở hai trường GP và MS")
T = [85, 70, 30]                                      # mục tiêu GIẢ ĐỊNH cho KR1–KR3

mean_g3   = df["G3"].mean()
pass_rate = df["passed"].mean() * 100
no_drop   = (df["G3"] >= df["G1"]).mean() * 100
good      = (df["G3"] >= 14).mean() * 100

kpi(fig, 0, "KPI", "Điểm TB G3", f"{mean_g3:.1f}", "thang 0–20, cả hai môn")
kpi(fig, 1, "KR1", "Tỷ lệ đạt", f"{pass_rate:.1f}%", f"G3 ≥ 10 · mục tiêu ≥ {T[0]}%")
kpi(fig, 2, "KR2", "Không tụt điểm", f"{no_drop:.1f}%", f"G3 ≥ G1 · mục tiêu ≥ {T[1]}%")
kpi(fig, 3, "KR3", "Điểm khá", f"{good:.1f}%", f"G3 ≥ 14 · mục tiêu ≥ {T[2]}%")

# 01 Line chart: xu hướng điểm qua 3 mốc
df.groupby("subject")[["G1", "G2", "G3"]].mean().T.plot(kind="line", marker="o", ax=ax[0])
ttl(ax[0], "01", "Điểm TB qua các kỳ", "Line chart · điểm đổi thế nào từ G1 đến G3?")
ax[0].set_ylabel("Điểm TB")

# 02 Histogram: hình dạng phân bố G3
df["G3"].plot(kind="hist", bins=20, range=(0, 20), color=BLUE, edgecolor="white", ax=ax[1])
ax[1].axvline(10, color="red", ls="--")
ttl(ax[1], "02", "Phân bố điểm cuối kỳ G3", "Histogram · lệch trái hay phải? Bao nhiêu em dưới 10?")
ax[1].set_xlabel("G3"); ax[1].set_ylabel("Số lượt học sinh–môn")

# 03 Box plot: so sánh phân bố theo môn × trường
df.boxplot(column="G3", by=["subject", "school"], ax=ax[2])
fig.suptitle("")                                      # bỏ tiêu đề tự sinh của boxplot(by=...)
ttl(ax[2], "03", "G3 theo môn và trường", "Box plot · nhóm nào lệch hoặc phân tán hơn?")
ax[2].set_xlabel("(môn, trường)"); ax[2].set_ylabel("G3")

# 04 Theo dõi KR
kr_bars(ax[3], ["KR1 Tỷ lệ đạt", "KR2 Không tụt điểm", "KR3 Điểm khá"], [pass_rate, no_drop, good], T)
ttl(ax[3], "04", "Theo dõi Key Results", "Thanh ngang · so với mục tiêu giả định")

fig.text(0.04, 0.02, "Định nghĩa: Đạt = G3 ≥ 10 · 1 dòng = 1 học sinh trong 1 môn (Toán 395 + Bồ Đào Nha 649) · Mục tiêu KR là giả định",
         fontsize=8, color="gray")
plt.show()
```

### 3.6. Cách đọc → Insight → Hành động

| Quan sát cần kiểm tra | Nếu thấy… | Insight có thể viết | Hành động đề xuất |
|---|---|---|---|
| 01: đường G1→G3 | Đi xuống | Điểm giảm dần theo kỳ, chương trình khó dần hoặc học sinh mất động lực | Ôn tập củng cố giữa kỳ 2 |
| 01: so hai môn | Một môn thấp hơn rõ | Môn đó cần ưu tiên nguồn lực | Phân bổ giờ phụ đạo cho môn đó |
| 02: cột ở 0 | Có cột đột biến | Một nhóm học sinh có thể bỏ thi hoặc bỏ học | Rà soát danh sách G3 = 0 |
| 03: box plot | Hộp hẹp, hộp rộng | Nhóm hộp rộng chênh lệch năng lực lớn | Chia lớp theo trình độ |

### 3.7. Dòng dùng cho Portfolio

- **Vấn đề:** hai trường cần theo dõi chất lượng học tập theo ba mốc điểm.
- **Phát hiện:** [điền: điểm đổi thế nào qua kỳ, môn nào thấp hơn].
- **Đề xuất:** [điền: ôn tập hoặc phụ đạo ở môn/trường nào].
- **Giới hạn:** dữ liệu cắt ngang, hai môn khác độ khó.

---

## 4. DASHBOARD 2 – STUDENT RISK MONITORING (CẢNH BÁO NGUY CƠ)

### 4.1. Bối cảnh nghiệp vụ

Giáo viên chủ nhiệm / cố vấn học tập cần một **danh sách ưu tiên** học sinh có khả năng trượt, ngay sau kỳ giữa năm, để can thiệp trước kỳ cuối. Nguồn lực có hạn nên không thể kèm tất cả học sinh.

### 4.2. Goal – Metric – Key Result

| Thành phần | Nội dung |
|---|---|
| **Goal** | Phát hiện sớm học sinh có nguy cơ trượt để can thiệp trước kỳ cuối |
| **KPI** | Tỷ lệ học sinh nhóm nguy cơ cao |
| **KR1** | Bắt được ≥ 85% học sinh trượt thật (độ phủ, recall) |
| **KR2** | ≥ 60% học sinh nhóm Cao thực sự trượt (độ chính xác, precision) |
| **KR3** | ≥ 60% học sinh thuộc nhóm Thấp (an toàn) |

### 4.3. Kịch bản phân tích

| Tiêu chí | Quyết định |
|---|---|
| Nhu cầu nghiệp vụ | Cần quy tắc **đơn giản, giải thích được** cho giáo viên |
| Dữ liệu sẵn có | G2, `failures`, `absences` có trước kỳ cuối |
| Năng lực tổ chức | Giáo viên đếm tín hiệu bằng tay được, không cần mô hình ML |
| Giới hạn | G2 liên quan rất mạnh tới G3, nên độ phủ cao một phần là hiển nhiên. Giá trị thực của dashboard nằm ở các ca "lọt lưới" |

### 4.4. Bảng chọn biểu đồ

| # | Câu hỏi | Biến và kiểu | Biểu đồ | Vì sao |
|---|---|---|---|---|
| 01 | Mỗi nhóm nguy cơ có bao nhiêu học sinh, theo môn? | `risk` (thứ tự), `subject` | Column ghép | So sánh số lượng giữa nhóm phân loại |
| 02 | Quy tắc có đáng tin không? Tỷ lệ trượt thực tế từng nhóm? | `risk` → % `fail` | Column + nhãn % | **Kiểm chứng** quy tắc bằng kết quả thật |
| 03 | G2 liên quan G3 ra sao? Ai "lọt lưới"? | G2, G3 (định lượng), `failures` | Scatter | Quan hệ hai biến định lượng, tìm điểm bất thường |
| 04 | KR đạt đến đâu? | tỷ lệ % và ngưỡng | Thanh ngang | Tiến độ so mục tiêu |

### 4.5. Code

```python
fig, ax = canvas("Student Risk Monitoring Dashboard", "Phát hiện sớm học sinh có nguy cơ trượt để can thiệp trước kỳ cuối")
T = [85, 60, 60]

share     = df["risk"].value_counts(normalize=True) * 100
fail_by   = df.groupby("risk", observed=True)["fail"].mean() * 100     # tỷ lệ trượt THỰC TẾ từng nhóm
warned    = df["risk"] != "Thấp"
recall    = (warned & df["fail"]).sum() / df["fail"].sum() * 100
precision = df.loc[df["risk"] == "Cao", "fail"].mean() * 100

kpi(fig, 0, "KPI", "Nhóm nguy cơ cao", f"{share['Cao']:.1f}%", "trên tổng lượt học sinh–môn")
kpi(fig, 1, "KR1", "Độ phủ cảnh báo", f"{recall:.1f}%", f"trượt thật nằm trong nhóm TB+Cao · mục tiêu ≥ {T[0]}%")
kpi(fig, 2, "KR2", "Độ chính xác nhóm Cao", f"{precision:.1f}%", f"nhóm Cao thực sự trượt · mục tiêu ≥ {T[1]}%")
kpi(fig, 3, "KR3", "Nhóm an toàn", f"{share['Thấp']:.1f}%", f"nhóm Thấp · mục tiêu ≥ {T[2]}%")

# 01 Column ghép: số lượng theo nhóm nguy cơ và môn
pd.crosstab(df["risk"], df["subject"]).plot(kind="bar", rot=0, ax=ax[0])
ttl(ax[0], "01", "Học sinh theo nhóm nguy cơ", "Column ghép · mỗi nhóm có bao nhiêu học sinh?")
ax[0].set_xlabel(""); ax[0].set_ylabel("Số lượt học sinh–môn")

# 02 Column: kiểm chứng quy tắc bằng tỷ lệ trượt thực tế
fail_by.plot(kind="bar", rot=0, color=[BLUE, GOLD, "#C0504D"], ax=ax[1])
ax[1].bar_label(ax[1].containers[0], fmt="%.1f%%")
ttl(ax[1], "02", "Tỷ lệ trượt thực tế theo nhóm", "Column · quy tắc nguy cơ có phân biệt được không?")
ax[1].set_xlabel(""); ax[1].set_ylabel("% trượt (G3 < 10)"); ax[1].set_ylim(0, 100)

# 03 Scatter: G2 vs G3, màu theo số lần trượt trước
df.plot.scatter(x="G2", y="G3", c="failures", colormap="viridis", alpha=0.4, ax=ax[2])
ax[2].axhline(10, color="red", ls="--", lw=1); ax[2].axvline(10, color="red", ls="--", lw=1)
ttl(ax[2], "03", "G2 so với G3", "Scatter · quan hệ thế nào? Ai khá ở G2 nhưng tụt về 0 ở G3?")

# 04 Theo dõi KR
kr_bars(ax[3], ["KR1 Độ phủ", "KR2 Độ chính xác", "KR3 Nhóm an toàn"], [recall, precision, share["Thấp"]], T)
ttl(ax[3], "04", "Theo dõi Key Results", "Thanh ngang · so với mục tiêu giả định")

fig.text(0.04, 0.02, "Định nghĩa: điểm nguy cơ = (G2<10) + (failures≥1) + (absences > percentile 75); 0 = Thấp, 1 = Trung bình, ≥2 = Cao · Trượt = G3 < 10",
         fontsize=8, color="gray")
plt.show()
```

### 4.6. Cách đọc → Insight → Hành động

| Quan sát | Nếu thấy… | Insight | Hành động |
|---|---|---|---|
| 02: tỷ lệ trượt tăng dần Thấp → TB → Cao | Có | Quy tắc phân biệt được nhóm, **đáng dùng** | Giao giáo viên chủ nhiệm theo dõi nhóm Cao trước |
| 02: nhóm Thấp vẫn trượt đáng kể | Có | Quy tắc bỏ sót một phần | Bổ sung tín hiệu (ví dụ nhân tố gia đình) |
| 03: điểm G2 ≥ 10 nhưng G3 ≈ 0 | Có | Có ca rơi tự do mà G2 không báo trước | Theo dõi vắng mặt và liên lạc gia đình ở giữa kỳ 2 |
| KR1 thấp hơn mục tiêu | Có | Cảnh báo chưa đủ rộng | Hạ ngưỡng cảnh báo, chấp nhận nhiều cảnh báo giả hơn |

> **Đánh đổi (nói khi bảo vệ):** hạ ngưỡng thì bắt được nhiều học sinh trượt hơn (recall tăng) nhưng giáo viên phải theo dõi nhiều em hơn (precision giảm). Chọn bên nào phụ thuộc nguồn lực của trường.

### 4.7. Dòng dùng cho Portfolio

- **Vấn đề:** trường cần danh sách ưu tiên học sinh có nguy cơ trượt, trước kỳ cuối.
- **Cách làm:** quy tắc đếm tín hiệu sớm, **kiểm chứng ngược** bằng kết quả thực tế (không dùng mô hình hộp đen).
- **Phát hiện:** [điền: nhóm Cao trượt …% so với nhóm Thấp …%].
- **Đề xuất:** [điền: ai can thiệp, khi nào, với nhóm nào].

---

## 5. DASHBOARD 3 – ATTENDANCE & STUDY HABITS (CHUYÊN CẦN VÀ THÓI QUEN)

### 5.1. Bối cảnh nghiệp vụ

Cố vấn học tập muốn hiểu học sinh đang vắng bao nhiêu và học bao lâu mỗi tuần, để biết nên tư vấn về thói quen tự học hay về việc đi học đều.

### 5.2. Goal – Metric – Key Result

| Thành phần | Nội dung |
|---|---|
| **Goal** | Nâng cao chuyên cần và thói quen tự học |
| **KPI** | Số buổi vắng trung bình |
| **KR1** | ≥ 35% học sinh tự học trên 5 giờ/tuần (`studytime ≥ 3`) |
| **KR2** | ≥ 80% học sinh vắng không quá 5 buổi |
| **KR3** | ≥ 75% học sinh uống rượu cuối tuần ở mức thấp (`Walc ≤ 2`) |

### 5.3. Kịch bản phân tích

| Tiêu chí | Quyết định |
|---|---|
| Nhu cầu nghiệp vụ | Hiểu hai thói quen: đi học (vắng) và tự học |
| Dữ liệu sẵn có | **Không có chuyên cần theo ngày.** Dùng `absences` (số đếm) và `studytime` (mã 1–4) làm biến đại diện |
| Năng lực tổ chức | Chỉ cần báo cáo theo kỳ |
| Giới hạn | Phải nêu trong báo cáo: không trả lời được "học vào thứ mấy" |

### 5.4. Bảng chọn biểu đồ

| # | Câu hỏi | Biến và kiểu | Biểu đồ | Vì sao |
|---|---|---|---|---|
| 01 | Học sinh tự học bao lâu mỗi tuần? | `studytime` (thứ tự 1–4) | Column có thứ tự | Biến thứ tự ít mức → column, **không** dùng histogram |
| 02 | Số buổi vắng phân bố ra sao? | `absences` (số đếm lệch phải) | Histogram | Xem hình dạng và đuôi dài |
| 03 | Điểm G3 trung bình theo mức vắng × mức tự học? | `absences` (nhóm) × `studytime` → G3 | Heatmap ma trận | Hai biến phân loại, một giá trị trung bình |
| 04 | KR đạt đến đâu? | tỷ lệ % và ngưỡng | Thanh ngang | Tiến độ |

### 5.5. Code

```python
fig, ax = canvas("Attendance & Study Habits Dashboard", "Nâng cao chuyên cần và thói quen tự học")
T = [35, 80, 75]

study_hi = (df["studytime"] >= 3).mean() * 100
abs_low  = (df["absences"] <= 5).mean() * 100
walc_low = (df["Walc"] <= 2).mean() * 100

kpi(fig, 0, "KPI", "Buổi vắng TB", f"{df['absences'].mean():.1f}", f"median = {df['absences'].median():.0f} · lệch phải")
kpi(fig, 1, "KR1", "Tự học > 5h/tuần", f"{study_hi:.1f}%", f"studytime ≥ 3 · mục tiêu ≥ {T[0]}%")
kpi(fig, 2, "KR2", "Vắng ≤ 5 buổi", f"{abs_low:.1f}%", f"mục tiêu ≥ {T[1]}%")
kpi(fig, 3, "KR3", "Uống cuối tuần thấp", f"{walc_low:.1f}%", f"Walc ≤ 2 · mục tiêu ≥ {T[2]}%")

# 01 Column có thứ tự: studytime là mã 1–4 → đổi nhãn rồi vẽ
sm = dict(zip(range(1, 5), ["<2h", "2–5h", "5–10h", ">10h"]))
df["studytime"].value_counts().sort_index().rename(sm).plot(kind="bar", rot=0, color=BLUE, ax=ax[0])
ttl(ax[0], "01", "Thời gian tự học mỗi tuần", "Column · học sinh tự học bao lâu?")
ax[0].set_xlabel(""); ax[0].set_ylabel("Số lượt học sinh–môn")

# 02 Histogram: gộp đuôi ≥ 30 để thấy phần thân
df["absences"].clip(upper=30).plot(kind="hist", bins=15, color=BLUE, edgecolor="white", ax=ax[1])
ax[1].axvline(df["absences"].median(), color="red", ls="--")
ttl(ax[1], "02", "Phân bố số buổi vắng", "Histogram · lệch phải? (đuôi ≥ 30 gộp lại, đường đỏ = median)")
ax[1].set_xlabel("Số buổi vắng"); ax[1].set_ylabel("Số lượt học sinh–môn")

# 03 Heatmap: G3 TB theo nhóm vắng × mức tự học
df["abs_grp"] = pd.cut(df["absences"], [-1, 0, 5, 10, 200], labels=["0", "1–5", "6–10", ">10"])
pt = df.pivot_table(values="G3", index="abs_grp", columns="studytime", aggfunc="mean", observed=False)
heat(ax[2], pt)
ttl(ax[2], "03", "G3 trung bình: số buổi vắng × mức tự học", "Heatmap · tổ hợp nào có điểm thấp?")
ax[2].set_xlabel("studytime (1 = <2h … 4 = >10h)"); ax[2].set_ylabel("Số buổi vắng")

# 04 Theo dõi KR
kr_bars(ax[3], ["KR1 Tự học >5h", "KR2 Vắng ≤5", "KR3 Rượu cuối tuần thấp"], [study_hi, abs_low, walc_low], T)
ttl(ax[3], "04", "Theo dõi Key Results", "Thanh ngang · so với mục tiêu giả định")

fig.text(0.04, 0.02, "Giới hạn: dữ liệu KHÔNG có chuyên cần theo ngày; dùng absences và studytime (mã 1–4) làm biến đại diện · Mục tiêu KR là giả định",
         fontsize=8, color="gray")
plt.show()
```

### 5.6. Cách đọc → Insight → Hành động

| Quan sát | Nếu thấy… | Insight | Hành động |
|---|---|---|---|
| 01: phần lớn ở mức <2h và 2–5h | Có | Tự học còn ít | Hướng dẫn kế hoạch tự học, câu lạc bộ học nhóm |
| 02: lệch phải, median thấp hơn mean | Có | Đa số vắng ít, một nhóm nhỏ vắng rất nhiều | Theo dõi **nhóm đuôi**, không dùng mean để mô tả |
| 03: hàng vắng = 0 có điểm **thấp** (ngược trực giác) | Có | Có thể là vấn đề **dữ liệu**: học sinh G3 = 0 có thể không được ghi nhận vắng | Kiểm tra bằng `df[df["G3"] == 0]["absences"].describe()` trước khi kết luận |
| 03: ô có ít học sinh | Có | Trung bình ô đó kém tin cậy | Nêu rõ cỡ mẫu, không suy diễn |

### 5.7. Dòng dùng cho Portfolio

- **Vấn đề:** hiểu thói quen học và chuyên cần của học sinh khi dữ liệu không có điểm danh theo ngày.
- **Cách làm:** chọn biến đại diện, nói rõ giới hạn, kiểm tra tính hợp lý của dữ liệu (ví dụ `G3 = 0` và vắng = 0).
- **Phát hiện:** [điền].
- **Đề xuất:** [điền: tư vấn cho nhóm nào].

---

## 6. DASHBOARD 4 – EDUCATIONAL SUPPORT (HỖ TRỢ HỌC TẬP)

> Đây là dashboard thầy đã làm mẫu trên lớp. **Đừng chép lại nguyên xi.** Hãy tự định nghĩa nhóm nguy cơ, tự đặt KR và giải thích được từng lựa chọn.

### 6.1. Bối cảnh nghiệp vụ

Phòng công tác học sinh cần biết **hỗ trợ học tập đang đến được ai**: có tập trung vào nhóm nguy cơ cao không, hình thức nào phổ biến, và nhóm được hỗ trợ có kết quả ra sao.

### 6.2. Goal – Metric – Key Result

| Thành phần | Nội dung |
|---|---|
| **Goal** | Tăng độ phủ hỗ trợ học tập, ưu tiên nhóm nguy cơ cao |
| **KPI** | Tỷ lệ nhóm nguy cơ cao |
| **KR1** | ≥ 80% học sinh nhóm Cao có ít nhất một hình thức hỗ trợ (độ phủ nhóm Cao) |
| **KR2** | Tỷ lệ đạt của nhóm có hỗ trợ ≥ 80% |
| **KR3** | ≥ 75% học sinh có ít nhất một hình thức hỗ trợ |

### 6.3. Kịch bản phân tích

| Tiêu chí | Quyết định |
|---|---|
| Nhu cầu nghiệp vụ | Phân bổ hỗ trợ đúng đối tượng |
| Dữ liệu sẵn có | Ba hình thức: `schoolsup` (nhà trường), `famsup` (gia đình), `paid` (học thêm trả phí). Gộp thành `support_any` |
| Năng lực tổ chức | Quyết định phân bổ theo kỳ, không cần real-time |
| Giới hạn | Chỉ biết **có hay không**, không biết cường độ hay chất lượng hỗ trợ |

### 6.4. Bảng chọn biểu đồ

| # | Câu hỏi | Biến và kiểu | Biểu đồ | Vì sao |
|---|---|---|---|---|
| 01 | Mỗi nhóm nguy cơ có bao nhiêu em được và chưa được hỗ trợ? | `risk` × `support_any` | Column ghép | So sánh hai nhóm trong từng nhóm nguy cơ |
| 02 | Hình thức hỗ trợ nào phổ biến nhất? | 3 biến nhị phân → % | Thanh ngang | So sánh tỷ lệ giữa các hình thức, nhãn dài |
| 03 | Nhóm có hỗ trợ và không có, điểm G3 TB khác nhau thế nào, theo môn? | `support_any`, `subject` → G3 | Column ghép | So sánh trung bình giữa nhóm |
| 04 | KR đạt đến đâu? | tỷ lệ % | Thanh ngang | Tiến độ |

### 6.5. Code

```python
fig, ax = canvas("Educational Support Dashboard", "Tăng độ phủ hỗ trợ học tập, ưu tiên nhóm nguy cơ cao")
T = [80, 80, 75]

high       = df["risk"] == "Cao"
cover_high = df.loc[high, "support_any"].mean() * 100
pass_sup   = df.loc[df["support_any"], "passed"].mean() * 100
any_sup    = df["support_any"].mean() * 100

kpi(fig, 0, "KPI", "Nhóm nguy cơ cao", f"{high.mean() * 100:.1f}%", "trên tổng lượt học sinh–môn")
kpi(fig, 1, "KR1", "Độ phủ nhóm Cao", f"{cover_high:.1f}%", f"có ≥ 1 hình thức hỗ trợ · mục tiêu ≥ {T[0]}%")
kpi(fig, 2, "KR2", "Tỷ lệ đạt (có hỗ trợ)", f"{pass_sup:.1f}%", f"G3 ≥ 10 · mục tiêu ≥ {T[1]}%")
kpi(fig, 3, "KR3", "Hỗ trợ bất kỳ", f"{any_sup:.1f}%", f"mục tiêu ≥ {T[2]}%")

# 01 Column ghép: có / chưa hỗ trợ theo nhóm nguy cơ
(pd.crosstab(df["risk"], df["support_any"])
   .rename(columns={False: "Chưa hỗ trợ", True: "Có hỗ trợ"})
   .plot(kind="bar", rot=0, ax=ax[0]))
ttl(ax[0], "01", "Hỗ trợ theo nhóm nguy cơ", "Column ghép · hỗ trợ có đến đúng nhóm cần không?")
ax[0].set_xlabel(""); ax[0].set_ylabel("Số lượt học sinh–môn")

# 02 Thanh ngang: tỷ lệ từng hình thức hỗ trợ
(df[["schoolsup", "famsup", "paid"]].eq("yes").mean().mul(100)
   .rename({"schoolsup": "Nhà trường", "famsup": "Gia đình", "paid": "Học thêm trả phí"})
   .sort_values().plot(kind="barh", color=BLUE, ax=ax[1]))
ax[1].bar_label(ax[1].containers[0], fmt="%.1f%%")
ttl(ax[1], "02", "Phân bố hình thức hỗ trợ", "Thanh ngang · hình thức nào phổ biến nhất?")
ax[1].set_xlabel("% lượt học sinh–môn")

# 03 Column ghép: G3 TB có / không hỗ trợ, theo môn
(df.groupby(["subject", "support_any"])["G3"].mean().unstack()
   .rename(columns={False: "Chưa hỗ trợ", True: "Có hỗ trợ"})
   .plot(kind="bar", rot=0, ax=ax[2]))
ttl(ax[2], "03", "Hỗ trợ và điểm cuối kỳ", "Column ghép · nhóm có hỗ trợ điểm ra sao?")
ax[2].set_xlabel(""); ax[2].set_ylabel("G3 trung bình")

# 04 Theo dõi KR
kr_bars(ax[3], ["KR1 Độ phủ nhóm Cao", "KR2 Tỷ lệ đạt", "KR3 Hỗ trợ bất kỳ"], [cover_high, pass_sup, any_sup], T)
ttl(ax[3], "04", "Theo dõi Key Results", "Thanh ngang · so với mục tiêu giả định")

fig.text(0.04, 0.02, "Định nghĩa: Hỗ trợ bất kỳ = schoolsup, famsup hoặc paid = yes · Nhóm nguy cơ xem mục 2.3 · Chỉ quan sát tương quan, không kết luận nhân quả",
         fontsize=8, color="gray")
plt.show()
```

### 6.6. Cách đọc → Insight → Hành động

| Quan sát | Nếu thấy… | Insight | Hành động |
|---|---|---|---|
| 01: nhóm Cao vẫn có nhiều em "Chưa hỗ trợ" | Có | Hỗ trợ **chưa nhắm đúng đối tượng** | Ưu tiên danh sách nhóm Cao cho phụ đạo |
| 02: một hình thức chiếm đa số | Có | Phụ thuộc vào một kênh (ví dụ gia đình) | Mở rộng kênh nhà trường cho em thiếu hỗ trợ gia đình |
| 03: nhóm **có hỗ trợ** lại điểm **thấp hơn** | Có | **Đừng kết luận hỗ trợ vô ích.** Đây là thiên lệch chọn mẫu: học sinh yếu thường mới được hỗ trợ | Cần so sánh trước và sau, hoặc so cùng mức nguy cơ |

> **Cái bẫy lớn nhất của dashboard này:** so trực tiếp nhóm có và không có hỗ trợ dễ cho kết luận ngược. Hãy nói rõ điều này khi bảo vệ.

### 6.7. Dòng dùng cho Portfolio

- **Vấn đề:** hỗ trợ học tập có đến đúng nhóm cần không?
- **Phát hiện:** [điền: độ phủ nhóm Cao …%, hình thức phổ biến nhất …].
- **Đề xuất:** [điền: ưu tiên nhóm nào, hình thức nào].
- **Giới hạn:** tương quan khác nhân quả, thiên lệch chọn mẫu.

---

## 7. DASHBOARD 5 – EDUCATIONAL EQUITY (CÔNG BẰNG GIÁO DỤC)

### 7.1. Bối cảnh nghiệp vụ

Ban giám hiệu / cơ quan quản lý cần kiểm tra **khoảng cách kết quả** giữa các nhóm học sinh (giới, khu vực, trường, internet, học vấn mẹ) để xem nhóm nào đang bị bỏ lại phía sau.

### 7.2. Goal – Metric – Key Result

| Thành phần | Nội dung |
|---|---|
| **Goal** | Thu hẹp khoảng cách kết quả giữa các nhóm học sinh |
| **KPI** | Khoảng cách điểm G3 TB lớn nhất giữa các nhóm (theo từng chiều) |
| **KR1** | Tỷ lệ đạt của học sinh nông thôn (R) ngang mức chung |
| **KR2** | Tỷ lệ đạt của học sinh không có internet ngang mức chung |
| **KR3** | Tỷ lệ đạt của học sinh có mẹ học vấn thấp (`Medu ≤ 1`) ngang mức chung |

> Mục tiêu KR ở đây là **bằng tỷ lệ đạt chung** (tiêu chí "ngang bằng"). Có cơ sở lý luận tự nhiên hơn một con số tuỳ ý.

### 7.3. Kịch bản phân tích

| Tiêu chí | Quyết định |
|---|---|
| Nhu cầu nghiệp vụ | Tìm chiều bất bình đẳng lớn nhất để ưu tiên chính sách |
| Dữ liệu sẵn có | `sex`, `address`, `school`, `internet`, `Medu` |
| Năng lực tổ chức | Quyết định cấp chính sách, báo cáo theo năm |
| Giới hạn | Chênh lệch có thể do yếu tố khác (hai trường khác quy mô). **Không nói nguyên nhân.** Nhóm nhỏ (ví dụ `Medu = 0`) kém tin cậy |

### 7.4. Bảng chọn biểu đồ

| # | Câu hỏi | Biến và kiểu | Biểu đồ | Vì sao |
|---|---|---|---|---|
| 01 | Chiều nào có khoảng cách điểm lớn nhất? | 5 biến phân loại → chênh lệch G3 TB | Thanh ngang | So sánh một độ đo giữa nhiều chiều |
| 02 | Đô thị/nông thôn × có/không internet: tỷ lệ đạt? | `address` × `internet` → % đạt | Heatmap | Hai biến phân loại, một giá trị |
| 03 | Học vấn mẹ liên quan thế nào đến G3? | `Medu` (thứ tự 0–4) → G3 TB | Column + cỡ mẫu n | Biến thứ tự ít mức; cỡ mẫu nhỏ phải hiện ra |
| 04 | KR đạt đến đâu? | tỷ lệ % và ngưỡng | Thanh ngang | Tiến độ |

### 7.5. Code

```python
fig, ax = canvas("Educational Equity Dashboard", "Thu hẹp khoảng cách kết quả giữa các nhóm học sinh")

df["edu_low"] = df["Medu"] <= 1                       # mẹ học vấn thấp: không học hoặc tiểu học
overall = df["pass_pct"].mean()
def pr(mask): return df.loc[mask, "pass_pct"].mean()  # tỷ lệ đạt của một nhóm
rural, no_net, low_edu = pr(df["address"] == "R"), pr(df["internet"] == "no"), pr(df["edu_low"])
T = [overall] * 3                                     # mục tiêu = ngang mức chung

def gap_of(d):                                        # chênh lệch G3 TB giữa nhóm cao nhất và thấp nhất
    m = df.groupby(d)["G3"].mean(); return m.max() - m.min()
gap = pd.Series({d: gap_of(d) for d in ["sex", "address", "school", "internet", "edu_low"]})

kpi(fig, 0, "KPI", "Khoảng cách G3 lớn nhất", f"{gap.max():.2f}", f"điểm · chiều: {gap.idxmax()}")
kpi(fig, 1, "KR1", "Đạt: nông thôn (R)", f"{rural:.1f}%", f"mục tiêu = mức chung {overall:.1f}%")
kpi(fig, 2, "KR2", "Đạt: không internet", f"{no_net:.1f}%", f"mục tiêu = mức chung {overall:.1f}%")
kpi(fig, 3, "KR3", "Đạt: mẹ học vấn thấp", f"{low_edu:.1f}%", f"mục tiêu = mức chung {overall:.1f}%")

# 01 Thanh ngang: khoảng cách theo chiều
gap.sort_values().plot(kind="barh", color=BLUE, ax=ax[0])
ax[0].bar_label(ax[0].containers[0], fmt="%.2f")
ttl(ax[0], "01", "Khoảng cách điểm G3 theo từng chiều", "Thanh ngang · chiều nào chênh lệch nhất?")
ax[0].set_xlabel("Chênh lệch G3 TB giữa nhóm cao và thấp (điểm)")

# 02 Heatmap: tỷ lệ đạt theo khu vực × internet
heat(ax[1], df.pivot_table(values="pass_pct", index="address", columns="internet", aggfunc="mean"))
ttl(ax[1], "02", "Tỷ lệ đạt (%): khu vực × internet", "Heatmap · nhóm nào thấp nhất?")
ax[1].set_xlabel("internet"); ax[1].set_ylabel("address (U đô thị, R nông thôn)")

# 03 Column: G3 TB theo học vấn mẹ, kèm cỡ mẫu n
g = df.groupby("Medu")["G3"].agg(["mean", "count"])
g["mean"].plot(kind="bar", rot=0, color=BLUE, ax=ax[2])
for i, n in enumerate(g["count"]):
    ax[2].text(i, g["mean"].iloc[i], f"n={n}", ha="center", va="bottom", fontsize=8)
ttl(ax[2], "03", "G3 trung bình theo học vấn mẹ", "Column · học vấn mẹ cao hơn, điểm cao hơn? (n = cỡ mẫu)")
ax[2].set_xlabel("Medu (0 không học … 4 đại học)"); ax[2].set_ylabel("G3 trung bình")

# 04 Theo dõi KR (ngang mức chung = 1)
kr_bars(ax[3], ["KR1 Nông thôn", "KR2 Không internet", "KR3 Mẹ học vấn thấp"], [rural, no_net, low_edu], T)
ttl(ax[3], "04", "Theo dõi Key Results", "Thanh ngang · so với mức chung của toàn bộ học sinh")

fig.text(0.04, 0.02, "Định nghĩa: Đạt = G3 ≥ 10 · Mẹ học vấn thấp = Medu ≤ 1 · Mô tả chênh lệch, KHÔNG kết luận nguyên nhân · Nhóm nhỏ kém tin cậy",
         fontsize=8, color="gray")
plt.show()
```

### 7.6. Cách đọc → Insight → Hành động

| Quan sát | Nếu thấy… | Insight | Hành động |
|---|---|---|---|
| 01: một chiều vượt trội | Có | Đó là chiều cần ưu tiên chính sách | Dồn nguồn lực cho nhóm thiệt thòi ở chiều đó |
| 01: chiều nào gần 0 | Có | Chiều đó tương đối công bằng | Không cần can thiệp riêng |
| 02: ô thấp nhất | Có | Nhóm chồng hai bất lợi (ví dụ nông thôn và không internet) | Hỗ trợ thiết bị hoặc điểm học chung |
| 03: cột `n` rất nhỏ | Có | Trung bình cột đó không tin cậy | Nêu rõ, không kết luận |
| 04: KR dưới mức chung | Có | Nhóm đang tụt lại so mặt bằng | Đặt mục tiêu rút ngắn khoảng cách theo kỳ |

### 7.7. Dòng dùng cho Portfolio

- **Vấn đề:** những nhóm học sinh nào đang bị bỏ lại phía sau?
- **Cách làm:** đo khoảng cách theo nhiều chiều, đặt mục tiêu "ngang mức chung", luôn hiện cỡ mẫu.
- **Phát hiện:** [điền: chiều chênh lệch nhất là …, nhóm thấp nhất là …].
- **Đề xuất:** [điền: chính sách hướng vào nhóm nào].

---

## 8. ĐỐI CHIẾU BÀI TẬP SLIDE VÀ CHECKLIST NỘP BÀI

### 8.1. Các bài tập trên slide nằm ở đâu trong 5 dashboard

| Bài tập trong slide | Nằm ở | Ghi chú |
|---|---|---|
| Line/Bar 1: G1, G2, G3 đổi thế nào | D1 – 01 | |
| Line/Bar 2: nam và nữ qua các kỳ | Bài mở rộng | Đổi `groupby("subject")` thành `groupby("sex")` ở D1 – 01 |
| Line/Bar 3: học theo ngày trong tuần | **Không làm được** | Dữ liệu không có. Nêu ở mục 1.2 và D3 |
| Line/Bar 4–6: G3 theo giới, trường, khu vực, internet | D1 – 03, D5 – 01, 02 | |
| Line/Bar 7: số học sinh theo `schoolsup` | D4 – 02 | |
| Histogram 1: dạng phân bố G3 | D1 – 02 | |
| Histogram 3: phân bố `studytime` | D3 – 01 | Dùng column vì biến thứ tự |
| Histogram 2, 4–8: tuổi, so sánh nhóm | Bài mở rộng | Dùng `df.boxplot(by=...)` hoặc hai histogram chồng |
| Heatmap 1: trường × giới | Bài mở rộng | `pivot_table(index="school", columns="sex")` + `heat()` |
| Heatmap 2: đô thị/nông thôn × internet | D5 – 02 | |
| Heatmap 3: `schoolsup` × `famsup` | Bài mở rộng | Thay vào D4 nếu thầy yêu cầu |
| Heatmap 5: học thời gian × số lần trượt | Bài mở rộng | `index="failures", columns="studytime"` |

### 8.2. Lưu hình

```python
fig.savefig("d1_academic.png", dpi=150, bbox_inches="tight")   # đặt trước plt.show()
```

### 8.3. Checklist trước khi nộp

| ✔ | Hạng mục |
|---|---|
| ☐ | Mỗi dashboard có Goal, 1 KPI, 3 KR, ghi rõ KR là giả định |
| ☐ | Mỗi biểu đồ có số thứ tự, loại biểu đồ và câu hỏi nghiệp vụ trong tiêu đề |
| ☐ | Mỗi biểu đồ có "phiếu 1 phút" (mục 0.2) trong Markdown cell |
| ☐ | Chân trang mỗi dashboard có định nghĩa chỉ số |
| ☐ | Có Data Health Check và bảng giới hạn dữ liệu (mục 1.2) |
| ☐ | Có bảng Insight → Hành động với **số liệu thật của bạn** |
| ☐ | Chỉ dùng pandas, numpy, matplotlib |
| ☐ | Giải thích được từng dòng code bằng lời |
| ☐ | Không kết luận nhân quả ở bất kỳ đâu |
| ☐ | Chạy lại notebook từ đầu (Restart & Run All) không lỗi |

---

## 9. CÂU HỎI THẦY CÓ THỂ HỎI KHI BẢO VỆ

| Câu hỏi | Gợi ý trả lời |
|---|---|
| Vì sao chọn line chart cho G1→G3, không phải bar? | Ba mốc có thứ tự thời gian, mục đích là xu hướng, line thể hiện sự thay đổi liên tục tốt hơn |
| Vì sao `studytime` dùng column mà không dùng histogram? | Là biến thứ tự có 4 mức rời rạc, histogram dành cho biến liên tục chia khoảng |
| Nhóm nguy cơ định nghĩa thế nào? Sao không dùng G3? | Dùng tín hiệu sớm G2, `failures`, `absences`. G3 là kết quả, dùng nó sẽ là lập luận vòng tròn |
| Ngưỡng 10 lấy ở đâu? | Là mức đạt thông thường của thang 0–20; với `absences` lấy percentile 75 từ chính dữ liệu |
| Dữ liệu không có chuyên cần theo ngày, em làm thế nào? | Dùng `absences` và `studytime` làm biến đại diện, nêu rõ giới hạn |
| Mục tiêu KR lấy từ đâu? | Giả định, đặt cao hơn mức hiện tại vài điểm %; riêng Equity lấy mức chung làm chuẩn "ngang bằng" |
| Nhóm có hỗ trợ điểm thấp hơn, vậy hỗ trợ vô ích? | Không. Thiên lệch chọn mẫu: học sinh yếu mới được hỗ trợ. Cần so sánh cùng mức nguy cơ hoặc trước/sau |
| Một học sinh học cả hai môn có bị tính hai lần không? | Có, 1 dòng = 1 học sinh–môn. Em nêu rõ ở chân trang và dùng "lượt học sinh–môn" |
| Vì sao không dùng pie chart? | Không thể hiện xu hướng, khó so sánh các phần gần nhau |
| Tổ chức cần gì để dùng dashboard này? | Chỉ cần xuất bảng điểm và vắng mặt theo kỳ, chạy notebook, không cần hạ tầng real-time |
| Insight nào bạn tâm đắc nhất? | Chọn một ca có số liệu thật, nêu câu hỏi, biểu đồ, phát hiện, hành động và giới hạn |

---

*Hết. Bước tiếp theo: chạy theo thứ tự cell ở mục 2.6, điền số liệu thật vào các bảng "Insight → Hành động" và các dòng Portfolio.*
