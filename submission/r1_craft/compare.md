# So sánh L với R

Chỉ số L/R là thứ tự box cao ≥ H=40 trong từng frame, theo thứ tự XML; bắt đầu từ 1.
Box L trong ignore_region được báo IGNORE_SCOPE, không tính SPURIOUS.

## adasind_249480.jpg
## adasind_261480.jpg
- L2 mid SPURIOUS
- L3 mid SPURIOUS
- L6+R3 mid WRONG_CLASS
## adasind_265065.jpg
- L1 mid IGNORE_SCOPE
- L2 mid IGNORE_SCOPE
- L3 mid IGNORE_SCOPE
- L7 center SPURIOUS
- L9 center SPURIOUS
- L10+R8 mid BOX_GEOMETRY
- R7 mid MISSING

## Theo zone
| zone | n_ref | matched | missing | spurious |
|---|---|---|---|---|
| center | 4 | 4 | 0 | 2 |
| mid | 13 | 10 | 3 | 4 |
| edge | 0 | 0 | 0 | 0 |
