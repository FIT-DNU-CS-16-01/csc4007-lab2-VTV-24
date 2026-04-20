# Metrics Summary
- dataset: imdb
- vectorizer: tfidf
- model: linearsvm
- val_accuracy: 0.9126
- val_macro_f1: 0.9126
- test_accuracy: 0.9100
- test_macro_f1: 0.9100

## How to read the result on IMDB
- IMDB khá cân bằng nên accuracy và macro-F1 có thể gần nhau.
- Dù vậy vẫn cần báo cáo macro-F1 để giữ thói quen đánh giá đúng khi sang dữ liệu lệch lớp.

## Test per-class report
- class `negative`: precision=0.9184, recall=0.9000, f1=0.9091, support=2500.0
- class `positive`: precision=0.9020, recall=0.9200, f1=0.9109, support=2500.0
- class `macro avg`: precision=0.9102, recall=0.9100, f1=0.9100, support=5000.0
- class `weighted avg`: precision=0.9102, recall=0.9100, f1=0.9100, support=5000.0
