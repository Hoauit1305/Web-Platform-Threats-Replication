# Trace Verification

1.  `trace_matching.smt`

    - Chức năng: Định nghĩa invariant ở dạng SMT-LIB thô
  
2.  `trace_matching_macroexpand.el`

    - Chức năng:
      + Mở rộng các macro all!, any!, map!, filter! thành các hàm đệ quy define-fun-rec
      + Cho phép SMT-LIB được Z3 hiểu trực tiếp
    - Đầu vào: SMT-LIB thô từ trace_matching.smt
    - Đầu ra: SMT-LIB đã expand macro (đệ quy)

3.  `parse_trace.pl`

    - Chức năng:
      + Phân tích JSON trace thành danh sách `events`
      + Kết hợp dữ liệu trace với invariant đã expand
      + Tạo SMT-LIB hoàn chỉnh để đưa vào solver
    - Đầu vào:
      + JSON trace từ trình duyệt
      + SMT-LIB đã expand từ bước 2 (các invariant)
    - Đầu ra: SMT-LIB hoàn chỉnh, chứa cả invariant + dữ liệu trace

4.  `smtlib_compat.pl`

    - Chức năng:
      + Chuẩn hóa cú pháp SMT-LIB
      + Đảm bảo tương thích với Z3 hoặc cvc5
    - Đầu vào: SMT-LIB hoàn chỉnh từ parse_trace.pl
    - Đầu ra: SMT-LIB sẵn sàng cho solver

5.  `z3_wrapper.pl`

    - Chức năng:
      + Chạy Z3 song song, hỗ trợ push/pop
      + Kiểm tra SAT/UNSAT dựa trên invariant
    - Đầu vào: SMT-LIB chuẩn từ smtlib_compat.pl
    - Đầu ra:
      + SAT → trace tuân thủ invariant
      + UNSAT → trace vi phạm invariant

#

trace_matching.smt -> SMT-LIB (macro expand) -> + JSON Trace -> SMT-LIB chuẩn -> Z3 -> SAT/UNSAT
