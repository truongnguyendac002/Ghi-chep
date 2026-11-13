Luồng mua SKU có khuyến mãi thì xử lý thế nào?
1. BEGIN TRANSACTION: tạo Transaction PENDING_PAYMENT -> COMMIT luôn để idempotent key không sinh giao dịch mới, và nếu chưa lỗi thì xử lý lại giao dịch này tránh sinh dublicate giao dịch
2. BEGIN TRANSACTION: 
  LOCK mã SKU trong kho nếu là đơn SKU
  LOCK avaiable, helding của promo_program đó lại
  Check các điều kiện: thông tin request của đơn có hợp lệ không, ngân sách hiện tại có hết không (đây là lý do đi khóa ngân sách trước) -> lỗi thì roll back cả cụm
  đạt -> update avaiable, helding của promo_program , 
  giữ 1 code từ kho merchant để dùng cho gift khuyến mãi
  Lưu dữ liệu các bảng sau đó
  -> COMMIT

3. IPN về:
  BEGIN TRANSACTION : Luôn cần khóa theo cùng 1 thứ tự
  Lock transaction để tránh 2 thread xử lý cùng lúc ->  Nếu là đơn có SKU thì khóa kho mã trong SKU -> khóa promo_program, cập nhật  ngân sách -> Nếu đơn có sinh gift khuyến mãi, thì gift này cần lấy code trong stock_code, LOCK stock_code này lại, nhả
  COMMIT
