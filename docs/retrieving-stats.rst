======================================
Lấy thống kê thử nghiệm ở định dạng CSV (Retrieve test statistics in CSV format)
======================================

Bạn có thể muốn tiêu thụ kết quả Locust của mình thông qua một tệp CSV. Trong trường hợp này, có hai cách để làm điều này.

Đầu tiên, khi chạy Locust với giao diện web, bạn có thể lấy các tệp CSV dưới tab Tải xuống Dữ liệu.

Thứ hai, bạn có thể chạy Locust với một cờ sẽ định kỳ lưu bốn tệp CSV. Điều này đặc biệt hữu ích
nếu bạn có kế hoạch chạy Locust theo cách tự động với cờ ``--headless``:

.. code-block:: console

    $ locust -f examples/basic.py --csv example --headless -t10m

Các tệp sẽ được đặt tên là ``example_stats.csv``, ``example_failures.csv``, ``example_exceptions.csv`` và ``example_stats_history.csv``
(khi sử dụng ``--csv example``). Hai tệp đầu tiên sẽ chứa thống kê và thất bại cho toàn bộ
bài kiểm tra, với một hàng cho mỗi mục thống kê (điểm cuối URL) và một hàng tổng hợp. ``example_stats_history.csv``
sẽ có các hàng mới với thống kê *hiện tại* (cửa sổ trượt 10 giây) được thêm vào trong suốt
bài kiểm tra. Mặc định chỉ hàng Tổng hợp được thêm vào định kỳ vào lịch sử thống kê, nhưng nếu Locust được bắt đầu với
cờ ``--csv-full-history``, một hàng cho mỗi mục thống kê (và Tổng hợp) được thêm vào mỗi lần
thống kê được viết (mỗi 2 giây mặc định).

Bạn cũng có thể tùy chỉnh cách thường xuyên việc này được viết:

.. code-block:: python

    import locust.stats
    locust.stats.CSV_STATS_INTERVAL_SEC = 5 # mặc định là 1 giây
