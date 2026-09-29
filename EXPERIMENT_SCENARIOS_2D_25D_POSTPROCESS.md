# Hướng dẫn các kịch bản thực nghiệm 2D, 2.5D và hậu xử lý

Tài liệu này mô tả cách chạy các thực nghiệm chính:

- **2D**: mỗi mẫu đầu vào sử dụng 1 lát CT.
- **2.5D - 3 slices**: mỗi mẫu sử dụng 3 lát CT lân cận.
- **2.5D - 5 slices**: mỗi mẫu sử dụng 5 lát CT lân cận.
- **Hậu xử lý 3D** trên mask dự đoán.
- Áp dụng cho hai bộ dữ liệu:
  - **NSCLC-Radiomics**
  - **NSCLC-Radiogenomics**

---

## 1. Nguyên tắc cấu hình của repository

Repository hiện sử dụng mô hình **config-only**:

- Không truyền các tham số train/inference/post-processing bằng command line.
- Các tham số được khai báo trong biến `CFG` ở đầu các file Python.
- Sau khi train, cấu hình của lần chạy được ghi lại vào:

```text
experiments/<EXP_NAME>/config.json
```

> **Lưu ý quan trọng:** các file `experiments/*/config.json` là bản ghi cấu hình của experiment đã chạy. Train script **không tự động đọc** các file JSON này. Muốn chạy lại experiment, cần sửa `CFG` trong file train tương ứng.

Hai file train chính:

```text
trains/train_nsclc_radiomics.py
trains/train_nsclc_radiogenomics.py
```

Hai wrapper inference:

```text
inferences/infer_nsclc_radiomics.py
inferences/infer_nsclc_radiogenomics.py
```

Module inference dùng chung:

```text
inferences/infer_transunet.py
```

File hậu xử lý:

```text
inferences/postprocess_predictions.py
```

---

# 2. Chuẩn bị môi trường

Từ thư mục gốc của repository:

```bash
pip install -r requirements.txt
```

Các lệnh trong tài liệu này giả định terminal đang đứng tại:

```text
2.5D-Lung-Tumor-Segmentation_official/
```

Checkpoint được tạo ở:

```text
experiments/<EXP_NAME>/best.pt
experiments/<EXP_NAME>/last.pt
```

Inference ghi kết quả vào:

```text
output/<EXP_NAME>/test/
```

Trong đó:

```text
predictions/
per_case_metrics.csv
slice_metrics.csv
summary_metrics.json
```

---

# 3. Quy tắc thiết kế so sánh 2D / 2.5D

Nếu mục tiêu là so sánh ảnh hưởng của số lát đầu vào, nên giữ cố định toàn bộ hyperparameter và **chỉ thay đổi**:

```python
EXP_NAME
NUM_SLICES
```

Tương ứng:

| Kịch bản        | `NUM_SLICES` |
| --------------- | -----------: |
| 2D              |            1 |
| 2.5D - 3 slices |            3 |
| 2.5D - 5 slices |            5 |

Các tham số như learning rate, loss weight, batch size, sampling ratio, augmentation, threshold... nên được giữ giống nhau trong một thí nghiệm ablation công bằng.

Trong repository hiện có một số experiment lịch sử thay đổi thêm sampling/loss weight. Các giá trị đó được trình bày ở phần cuối tài liệu để phân biệt với protocol ablation được khuyến nghị.

---

# 4. Dataset 1 - NSCLC-Radiomics

## 4.1. File train

Sử dụng:

```bash
python trains/train_nsclc_radiomics.py
```

Các đường dẫn dữ liệu trong `CFG` phải trỏ tới:

```python
"PROCESSED_ROOT": str(ROOT_DIR / "data" / "nsclc-radiomics" / "processed"),
"NIFTI_ROOT": str(ROOT_DIR / "data" / "nsclc-radiomics" / "nifti"),
"TRAIN_SPLIT": str(ROOT_DIR / "data" / "nsclc-radiomics" / "config" / "train.txt"),
"VAL_SPLIT": str(ROOT_DIR / "data" / "nsclc-radiomics" / "config" / "val.txt"),
"SAVE_ROOT": str(ROOT_DIR / "experiments"),
```

Nên giữ nguyên các đường dẫn trên cho cả ba experiment 2D, 3-slice và 5-slice.

---

## 4.2. Thực nghiệm R1 - 2D

Trong:

```text
trains/train_nsclc_radiomics.py
```

đặt:

```python
"EXP_NAME": "nsclc-radiomics_2d",
"NUM_SLICES": 1,
```

Các tham số còn lại giữ nguyên.

Ví dụ phần cấu hình quan trọng:

```python
CFG = {
    "EXP_NAME": "nsclc-radiomics_2d",

    "PROCESSED_ROOT": str(ROOT_DIR / "data" / "nsclc-radiomics" / "processed"),
    "NIFTI_ROOT": str(ROOT_DIR / "data" / "nsclc-radiomics" / "nifti"),
    "TRAIN_SPLIT": str(ROOT_DIR / "data" / "nsclc-radiomics" / "config" / "train.txt"),
    "VAL_SPLIT": str(ROOT_DIR / "data" / "nsclc-radiomics" / "config" / "val.txt"),

    "SAVE_ROOT": str(ROOT_DIR / "experiments"),

    "NUM_SLICES": 1,

    # giữ nguyên các tham số model/training còn lại
}
```

Chạy:

```bash
python trains/train_nsclc_radiomics.py
```

Checkpoint tốt nhất:

```text
experiments/nsclc-radiomics_2d/best.pt
```

---

## 4.3. Thực nghiệm R2 - 2.5D, 3 slices

Đây là cấu hình gần với experiment:

```text
experiments/nsclc-radiomics_2.5d-3/
```

Đặt:

```python
"EXP_NAME": "nsclc-radiomics_2.5d-3",
"NUM_SLICES": 3,
```

Chạy:

```bash
python trains/train_nsclc_radiomics.py
```

Checkpoint:

```text
experiments/nsclc-radiomics_2.5d-3/best.pt
```

---

## 4.4. Thực nghiệm R3 - 2.5D, 5 slices

Đặt:

```python
"EXP_NAME": "nsclc-radiomics_2.5d-5",
"NUM_SLICES": 5,
```

Chạy:

```bash
python trains/train_nsclc_radiomics.py
```

Checkpoint:

```text
experiments/nsclc-radiomics_2.5d-5/best.pt
```

---

# 5. Inference trên NSCLC-Radiomics

File wrapper:

```text
inferences/infer_nsclc_radiomics.py
```

Cấu hình quan trọng:

```python
CFG = dict(BASE_CFG) | {
    "CHECKPOINT_PATH": ...,
    "SPLIT_PATH": ...,
    "PROCESSED_ROOT": ...,
    "NIFTI_ROOT": ...,
    "OUTPUT_DIR": ...,
    "NUM_SLICES": ...,
    "DEVICE": ...,
    "PIN_MEMORY": ...,
    "EXCLUDE_CASE": [],
}
```

`NUM_SLICES` trong inference **phải giống** `NUM_SLICES` khi train checkpoint.

Nếu không giống, `infer_transunet.py` sẽ phát hiện:

```text
checkpoint in_channels and CFG NUM_SLICES differ
```

---

## 5.1. Inference R1 - 2D

Sửa:

```python
"CHECKPOINT_PATH": r"experiments\nsclc-radiomics_2d\best.pt",

"OUTPUT_DIR": str(
    ROOT_DIR / "output" / "nsclc-radiomics_2d" / "test"
),

"NUM_SLICES": 1,
```

Giữ:

```python
"SPLIT_PATH": str(
    ROOT_DIR / "data" / "nsclc-radiomics" / "config" / "test.txt"
),

"PROCESSED_ROOT": str(
    ROOT_DIR / "data" / "nsclc-radiomics" / "processed"
),

"NIFTI_ROOT": str(
    ROOT_DIR / "data" / "nsclc-radiomics" / "nifti"
),
```

Chạy:

```bash
python inferences/infer_nsclc_radiomics.py
```

Output:

```text
output/nsclc-radiomics_2d/test/
```

---

## 5.2. Inference R2 - 2.5D, 3 slices

Sửa:

```python
"CHECKPOINT_PATH": r"experiments\nsclc-radiomics_2.5d-3\best.pt",

"OUTPUT_DIR": str(
    ROOT_DIR / "output" / "nsclc-radiomics_2.5d-3" / "test"
),

"NUM_SLICES": 3,
```

Chạy:

```bash
python inferences/infer_nsclc_radiomics.py
```

Output:

```text
output/nsclc-radiomics_2.5d-3/test/
```

---

## 5.3. Inference R3 - 2.5D, 5 slices

Sửa:

```python
"CHECKPOINT_PATH": r"experiments\nsclc-radiomics_2.5d-5\best.pt",

"OUTPUT_DIR": str(
    ROOT_DIR / "output" / "nsclc-radiomics_2.5d-5" / "test"
),

"NUM_SLICES": 5,
```

Chạy:

```bash
python inferences/infer_nsclc_radiomics.py
```

Output:

```text
output/nsclc-radiomics_2.5d-5/test/
```

---

# 6. Hậu xử lý NSCLC-Radiomics

Sau inference, các prediction NIfTI nằm ở:

```text
output/<EXP_NAME>/test/predictions/
```

File hậu xử lý:

```text
inferences/postprocess_predictions.py
```

Đặt:

```python
"DATASET": "nsclc_radiomics",
```

Ví dụ hậu xử lý experiment 2.5D - 5 slices:

```python
CFG = {
    "DATASET": "nsclc_radiomics",

    "PREDICTIONS_DIR": str(
        ROOT_DIR
        / "output"
        / "nsclc-radiomics_2.5d-5"
        / "test"
        / "predictions"
    ),

    "NIFTI_ROOT": str(
        ROOT_DIR
        / "data"
        / "nsclc-radiomics"
        / "nifti"
    ),

    "OUTPUT_DIR": str(
        ROOT_DIR
        / "output"
        / "nsclc-radiomics_2.5d-5"
        / "test"
        / "post_processed"
    ),

    "CLOSING_RADIUS_MM": 3.0,
    "OPENING_RADIUS_MM": 0.0,
    "MIN_COMPONENT_VOLUME_MM3": 50.0,
    "CONNECTIVITY": 26,
    "COMPONENT_LINK_DISTANCE_MM": 9.0,
    "SECONDARY_CLUSTER_MIN_VOLUME_RATIO": 0.10,

    "RADIOGENOMICS_VOLUME_WEIGHT": 0.20,
    "RADIOGENOMICS_Z_THICKNESS_WEIGHT": 0.10,
    "RADIOGENOMICS_SLICE_STABILITY_WEIGHT": 0.70,
}
```

Ba tham số `RADIOGENOMICS_*` không được sử dụng cho dataset Radiomics.

Chạy:

```bash
python inferences/postprocess_predictions.py
```

Output:

```text
output/nsclc-radiomics_2.5d-5/test/post_processed/
```

Trong đó:

```text
predictions/
per_case_metrics.csv
slice_metrics.csv
summary_metrics.json
```

Muốn hậu xử lý 2D hoặc 3-slice chỉ cần đổi:

```python
PREDICTIONS_DIR
OUTPUT_DIR
```

sang experiment tương ứng.

---

# 7. Dataset 2 - NSCLC-Radiogenomics

## 7.1. File train

Sử dụng:

```bash
python trains/train_nsclc_radiogenomics.py
```

Các đường dẫn dataset:

```python
"PROCESSED_ROOT": str(
    ROOT_DIR / "data" / "nsclc_radiogenomics" / "processed"
),

"NIFTI_ROOT": str(
    ROOT_DIR / "data" / "nsclc_radiogenomics" / "nifti"
),

"TRAIN_SPLIT": str(
    ROOT_DIR / "data" / "nsclc_radiogenomics" / "config" / "train.txt"
),

"VAL_SPLIT": str(
    ROOT_DIR / "data" / "nsclc_radiogenomics" / "config" / "val.txt"
),

"SAVE_ROOT": str(ROOT_DIR / "experiments"),
```

> Để có protocol train/validation/test rõ ràng, khuyến nghị dùng `val.txt` cho `VAL_SPLIT`. `test.txt` chỉ nên dùng ở inference cuối cùng.

---

## 7.2. Thực nghiệm G1 - 2D

Đặt:

```python
"EXP_NAME": "nsclc_radiogenomics_2d",
"NUM_SLICES": 1,
"RESUME_PATH": "",
"RESUME_WEIGHTS_ONLY": False,
```

Chạy:

```bash
python trains/train_nsclc_radiogenomics.py
```

Checkpoint:

```text
experiments/nsclc_radiogenomics_2d/best.pt
```

Repository hiện cũng có snapshot:

```text
experiments/nsclc_radiogenomics_2d/config.json
```

---

## 7.3. Thực nghiệm G2 - 2.5D, 3 slices

Đặt:

```python
"EXP_NAME": "nsclc_radiogenomics_2.5d-3",
"NUM_SLICES": 3,
"RESUME_PATH": "",
"RESUME_WEIGHTS_ONLY": False,
```

Chạy:

```bash
python trains/train_nsclc_radiogenomics.py
```

Checkpoint:

```text
experiments/nsclc_radiogenomics_2.5d-3/best.pt
```

Repository hiện có một experiment lịch sử gần tương ứng:

```text
experiments/nsclc_radiogenomics_2.5d-3_final/
```

---

## 7.4. Thực nghiệm G3 - 2.5D, 5 slices

Đặt:

```python
"EXP_NAME": "nsclc_radiogenomics_2.5d-5",
"NUM_SLICES": 5,
"RESUME_PATH": "",
"RESUME_WEIGHTS_ONLY": False,
```

Chạy:

```bash
python trains/train_nsclc_radiogenomics.py
```

Checkpoint:

```text
experiments/nsclc_radiogenomics_2.5d-5/best.pt
```

Repository hiện có experiment:

```text
experiments/nsclc_radiogenomics_2.5d-5_final/
```

Ngoài ra còn:

```text
experiments/nsclc_radiogenomics_2.5d-5_final_fullZ/
```

Đây là một run tiếp tục từ checkpoint:

```python
"RESUME_PATH":
    r"experiments\nsclc_radiogenomics_2.5d-5_final\best.pt"
```

Vì vậy không nên nhầm `*_final_fullZ` với một experiment 5-slice train từ đầu.

---

# 8. Inference trên NSCLC-Radiogenomics

File:

```text
inferences/infer_nsclc_radiogenomics.py
```

Wrapper hiện tại trong repo đang hướng tới checkpoint 5-slice.

---

## 8.1. Inference G1 - 2D

Sửa:

```python
"CHECKPOINT_PATH":
    r"experiments\nsclc_radiogenomics_2d\best.pt",

"OUTPUT_DIR": str(
    ROOT_DIR
    / "output"
    / "nsclc_radiogenomics_2d"
    / "test"
),

"NUM_SLICES": 1,
```

Giữ dataset:

```python
"SPLIT_PATH": str(
    ROOT_DIR / "data" / "nsclc_radiogenomics" / "config" / "test.txt"
),

"PROCESSED_ROOT": str(
    ROOT_DIR / "data" / "nsclc_radiogenomics" / "processed"
),

"NIFTI_ROOT": str(
    ROOT_DIR / "data" / "nsclc_radiogenomics" / "nifti"
),
```

Chạy:

```bash
python inferences/infer_nsclc_radiogenomics.py
```

---

## 8.2. Inference G2 - 2.5D, 3 slices

```python
"CHECKPOINT_PATH":
    r"experiments\nsclc_radiogenomics_2.5d-3\best.pt",

"OUTPUT_DIR": str(
    ROOT_DIR
    / "output"
    / "nsclc_radiogenomics_2.5d-3"
    / "test"
),

"NUM_SLICES": 3,
```

Chạy:

```bash
python inferences/infer_nsclc_radiogenomics.py
```

---

## 8.3. Inference G3 - 2.5D, 5 slices

```python
"CHECKPOINT_PATH":
    r"experiments\nsclc_radiogenomics_2.5d-5\best.pt",

"OUTPUT_DIR": str(
    ROOT_DIR
    / "output"
    / "nsclc_radiogenomics_2.5d-5"
    / "test"
),

"NUM_SLICES": 5,
```

Chạy:

```bash
python inferences/infer_nsclc_radiogenomics.py
```

Nếu dùng đúng checkpoint hiện có trong repository:

```python
"CHECKPOINT_PATH":
    r"experiments\nsclc_radiogenomics_2.5d-5_final\best.pt",

"OUTPUT_DIR": str(
    ROOT_DIR
    / "output"
    / "nsclc_radiogenomics_2.5d-5_final"
    / "test"
),

"NUM_SLICES": 5,
```

---

# 9. Hậu xử lý NSCLC-Radiogenomics

Đặt:

```python
"DATASET": "nsclc_radiogenomics",
```

Ví dụ cho 5-slice:

```python
CFG = {
    "DATASET": "nsclc_radiogenomics",

    "PREDICTIONS_DIR": str(
        ROOT_DIR
        / "output"
        / "nsclc_radiogenomics_2.5d-5"
        / "test"
        / "predictions"
    ),

    "NIFTI_ROOT": str(
        ROOT_DIR
        / "data"
        / "nsclc_radiogenomics"
        / "nifti"
    ),

    "OUTPUT_DIR": str(
        ROOT_DIR
        / "output"
        / "nsclc_radiogenomics_2.5d-5"
        / "test"
        / "post_processed"
    ),

    "CLOSING_RADIUS_MM": 3.0,
    "OPENING_RADIUS_MM": 0.0,
    "MIN_COMPONENT_VOLUME_MM3": 50.0,
    "CONNECTIVITY": 26,
    "COMPONENT_LINK_DISTANCE_MM": 9.0,
    "SECONDARY_CLUSTER_MIN_VOLUME_RATIO": 0.10,

    "RADIOGENOMICS_VOLUME_WEIGHT": 0.20,
    "RADIOGENOMICS_Z_THICKNESS_WEIGHT": 0.10,
    "RADIOGENOMICS_SLICE_STABILITY_WEIGHT": 0.70,
}
```

Chạy:

```bash
python inferences/postprocess_predictions.py
```

Với Radiogenomics, sau pipeline morphology/component filtering chuẩn, code còn gọi bước:

```text
retain_weighted_stable_lesion(...)
```

để chọn lesion theo điểm số kết hợp:

```text
volume
+ độ dày theo trục Z
+ độ ổn định giữa các slice
```

Trọng số mặc định:

```python
RADIOGENOMICS_VOLUME_WEIGHT = 0.20
RADIOGENOMICS_Z_THICKNESS_WEIGHT = 0.10
RADIOGENOMICS_SLICE_STABILITY_WEIGHT = 0.70
```

---

# 10. Pipeline hậu xử lý đang được sử dụng

Theo `postprocess_predictions.py`, pipeline tổng quát là:

```text
Prediction 3D
    ↓
3D binary closing
    ↓
Optional opening
    ↓
Hole filling
    ↓
Loại connected component quá nhỏ
    ↓
Gom component theo khoảng cách vật lý
    ↓
Giữ dominant cluster / secondary cluster phù hợp
    ↓
Radiogenomics only:
weighted stable lesion selection
    ↓
Final 3D mask
    ↓
Re-evaluation
```

Các tham số mặc định:

| Tham số                              | Giá trị |
| ------------------------------------ | ------: |
| `CLOSING_RADIUS_MM`                  |   `3.0` |
| `OPENING_RADIUS_MM`                  |   `0.0` |
| `MIN_COMPONENT_VOLUME_MM3`           |  `50.0` |
| `CONNECTIVITY`                       |    `26` |
| `COMPONENT_LINK_DISTANCE_MM`         |   `9.0` |
| `SECONDARY_CLUSTER_MIN_VOLUME_RATIO` |  `0.10` |

Opening mặc định bằng `0.0`, tức là bị vô hiệu hóa.

---

# 11. Không bật tuning trên test set cho kết quả chính của paper

File hậu xử lý còn có:

```python
TUNING_CFG = {
    "RUN_TUNING": False,
    ...
}
```

Giữ:

```python
"RUN_TUNING": False
```

cho kết quả test chính thức.

Nếu bật `RUN_TUNING=True`, code sẽ grid-search tham số hậu xử lý bằng ground-truth của chính tập đang đánh giá. Điều này gây **test-set leakage** và kết quả sẽ mang tính oracle/sensitivity analysis, không nên dùng làm kết quả chính.

Nếu cần tuning hậu xử lý, nên:

```text
train
  ↓
validation → chọn tham số post-processing
  ↓
freeze toàn bộ tham số
  ↓
test → đánh giá đúng một lần
```

---

# 12. Bảng tổng hợp sáu thực nghiệm chính

| ID  | Dataset             | Kiểu            | `NUM_SLICES` | Train script                          |
| --- | ------------------- | --------------- | -----------: | ------------------------------------- |
| R1  | NSCLC-Radiomics     | 2D              |            1 | `trains/train_nsclc_radiomics.py`     |
| R2  | NSCLC-Radiomics     | 2.5D - 3 slices |            3 | `trains/train_nsclc_radiomics.py`     |
| R3  | NSCLC-Radiomics     | 2.5D - 5 slices |            5 | `trains/train_nsclc_radiomics.py`     |
| G1  | NSCLC-Radiogenomics | 2D              |            1 | `trains/train_nsclc_radiogenomics.py` |
| G2  | NSCLC-Radiogenomics | 2.5D - 3 slices |            3 | `trains/train_nsclc_radiogenomics.py` |
| G3  | NSCLC-Radiogenomics | 2.5D - 5 slices |            5 | `trains/train_nsclc_radiogenomics.py` |

Sau mỗi run:

```text
train
  ↓
best.pt
  ↓
inference trên test.txt
  ↓
raw prediction NIfTI
  ↓
post-processing
  ↓
post-processed prediction NIfTI
  ↓
summary_metrics.json
```

---

# 13. Checklist chạy một experiment hoàn chỉnh

Ví dụ với:

```text
NSCLC-Radiomics + 2.5D 5 slices
```

### Bước 1 - sửa train config

Trong:

```text
trains/train_nsclc_radiomics.py
```

đặt:

```python
"EXP_NAME": "nsclc-radiomics_2.5d-5",
"NUM_SLICES": 5,
"RESUME_PATH": "",
```

### Bước 2 - train

```bash
python trains/train_nsclc_radiomics.py
```

Kiểm tra:

```text
experiments/nsclc-radiomics_2.5d-5/best.pt
```

### Bước 3 - sửa inference config

Trong:

```text
inferences/infer_nsclc_radiomics.py
```

đặt:

```python
"CHECKPOINT_PATH":
    r"experiments\nsclc-radiomics_2.5d-5\best.pt",

"OUTPUT_DIR": str(
    ROOT_DIR
    / "output"
    / "nsclc-radiomics_2.5d-5"
    / "test"
),

"NUM_SLICES": 5,
```

### Bước 4 - inference

```bash
python inferences/infer_nsclc_radiomics.py
```

Kiểm tra:

```text
output/nsclc-radiomics_2.5d-5/test/predictions/
output/nsclc-radiomics_2.5d-5/test/summary_metrics.json
```

### Bước 5 - sửa post-processing config

```python
"DATASET": "nsclc_radiomics",

"PREDICTIONS_DIR": str(
    ROOT_DIR
    / "output"
    / "nsclc-radiomics_2.5d-5"
    / "test"
    / "predictions"
),

"NIFTI_ROOT": str(
    ROOT_DIR
    / "data"
    / "nsclc-radiomics"
    / "nifti"
),

"OUTPUT_DIR": str(
    ROOT_DIR
    / "output"
    / "nsclc-radiomics_2.5d-5"
    / "test"
    / "post_processed"
),
```

### Bước 6 - chạy hậu xử lý

```bash
python inferences/postprocess_predictions.py
```

### Bước 7 - lấy kết quả cuối

```text
output/nsclc-radiomics_2.5d-5/test/post_processed/summary_metrics.json
```

---

# 14. Các config lịch sử hiện có trong repository

Repository hiện lưu các snapshot đáng chú ý:

```text
experiments/nsclc-radiomics_2.5d-3/config.json

experiments/nsclc_radiogenomics_2d/config.json
experiments/nsclc_radiogenomics_2.5d-3_final/config.json
experiments/nsclc_radiogenomics_2.5d-5_final/config.json
experiments/nsclc_radiogenomics_2.5d-5_final_fullZ/config.json

experiments/transunet_2d_micro_dice_loss_balanced_sampling/config.json
experiments/transunet_2.5d-3_balanced_sampling/config.json
experiments/transunet_2.5d-5_balanced_sampling/config.json
```

Một số config lịch sử **không chỉ thay đổi `NUM_SLICES`** mà còn thay đổi sampling hoặc loss weight.

Ví dụ Radiogenomics:

### `nsclc_radiogenomics_2d`

```python
NUM_SLICES = 1
HARD_NEGATIVE_RADIUS = 3
BALANCED_TRAIN_SAMPLING = True
POSITIVE_FRACTION = 0.4
HARD_NEGATIVE_FRACTION = 0.2
EASY_NEGATIVE_FRACTION = 0.4
BCE_POS_WEIGHT = 1.2
DICE_FOREGROUND_WEIGHT = 1.2
```

### `nsclc_radiogenomics_2.5d-3_final`

```python
NUM_SLICES = 3
BALANCED_TRAIN_SAMPLING = False
TRAIN_BATCHES_PER_EPOCH = 500
BCE_POS_WEIGHT = 1.2
DICE_FOREGROUND_WEIGHT = 1.2
```

### `nsclc_radiogenomics_2.5d-5_final`

```python
NUM_SLICES = 5
BALANCED_TRAIN_SAMPLING = True

POSITIVE_FRACTION = 0.35
HARD_NEGATIVE_FRACTION = 0.20
EASY_NEGATIVE_FRACTION = 0.45

TRAIN_BATCHES_PER_EPOCH = 500

BCE_POS_WEIGHT = 1.3
DICE_FOREGROUND_WEIGHT = 1.3
```

Vì vậy, nếu sử dụng trực tiếp kết quả các experiment lịch sử để kết luận rằng:

```text
2D < 2.5D-3 < 2.5D-5
```

thì cần lưu ý rằng khác biệt kết quả có thể đến từ cả:

```text
NUM_SLICES
sampling strategy
loss weighting
train split / validation split
resume strategy
```

chứ không chỉ riêng số lượng slice.

Đối với một ablation chính thức trong paper, nên chạy lại ba cấu hình với mọi tham số giống nhau ngoại trừ:

```python
NUM_SLICES = 1
NUM_SLICES = 3
NUM_SLICES = 5
```

---

# 15. Đề xuất cách đặt tên experiment thống nhất

Nên dùng:

```text
nsclc-radiomics_2d
nsclc-radiomics_2.5d-3
nsclc-radiomics_2.5d-5

nsclc_radiogenomics_2d
nsclc_radiogenomics_2.5d-3
nsclc_radiogenomics_2.5d-5
```

Kết quả tương ứng:

```text
experiments/
├── nsclc-radiomics_2d/
├── nsclc-radiomics_2.5d-3/
├── nsclc-radiomics_2.5d-5/
├── nsclc_radiogenomics_2d/
├── nsclc_radiogenomics_2.5d-3/
└── nsclc_radiogenomics_2.5d-5/
```

và:

```text
output/
├── nsclc-radiomics_2d/test/
├── nsclc-radiomics_2.5d-3/test/
├── nsclc-radiomics_2.5d-5/test/
├── nsclc_radiogenomics_2d/test/
├── nsclc_radiogenomics_2.5d-3/test/
└── nsclc_radiogenomics_2.5d-5/test/
```

Cách đặt tên này giúp bảng thực nghiệm trong paper, checkpoint, inference output và post-processing output ánh xạ một-một với nhau.

---

# 16. Tóm tắt lệnh chạy

## NSCLC-Radiomics

Train:

```bash
python trains/train_nsclc_radiomics.py
```

Inference:

```bash
python inferences/infer_nsclc_radiomics.py
```

Post-processing:

```bash
python inferences/postprocess_predictions.py
```

## NSCLC-Radiogenomics

Train:

```bash
python trains/train_nsclc_radiogenomics.py
```

Inference:

```bash
python inferences/infer_nsclc_radiogenomics.py
```

Post-processing:

```bash
python inferences/postprocess_predictions.py
```

Điểm khác nhau giữa các run nằm trong `CFG` của từng file, đặc biệt là:

```text
EXP_NAME
NUM_SLICES
CHECKPOINT_PATH
OUTPUT_DIR
DATASET
PREDICTIONS_DIR
NIFTI_ROOT
```
