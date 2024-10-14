.. _custom-load-shape:

==================
Hình dạng tải tùy chỉnh (Custom load shapes)
==================

Đôi khi cần một hình dạng tải hoàn toàn tùy chỉnh mà không thể đạt được bằng cách đơn giản là thiết lập hoặc thay đổi số lượng người dùng và tốc độ spawn. Ví dụ, bạn có thể muốn tạo ra một tải spike hoặc tăng giảm tại các thời điểm tùy chỉnh. Bằng cách sử dụng một lớp `LoadTestShape` bạn có toàn quyền kiểm soát số lượng người dùng và tốc độ spawn tại tất cả các thời điểm.

Định nghĩa một lớp kế thừa lớp LoadTestShape trong tệp locust của bạn. Nếu loại lớp này được tìm thấy thì nó sẽ tự động được sử dụng bởi Locust.

Trong lớp này, bạn xác định một phương thức `tick()` trả về một bộ giá trị với số lượng người dùng và tốc độ spawn mong muốn (hoặc `None` để dừng bài kiểm tra). Locust sẽ gọi phương thức `tick()` khoảng một lần mỗi giây.

Trong lớp bạn cũng có quyền truy cập vào phương thức `get_run_time()`, để kiểm tra bài kiểm tra đã chạy được bao lâu.

Ví dụ
-------

Lớp hình dạng này sẽ tăng số lượng người dùng theo từng khối 100 và sau đó dừng bài kiểm tra sau 10 phút:

.. code-block:: python

    class MyCustomShape(LoadTestShape):
        time_limit = 600
        spawn_rate = 20
        
        def tick(self):
            run_time = self.get_run_time()

            if run_time < self.time_limit:
                # Số lượng người dùng làm tròn đến hàng trăm gần nhất.
                user_count = round(run_time, -2)
                return (user_count, spawn_rate)

            return None

Chức năng này được thể hiện rõ hơn trong `ví dụ trên github <https://github.com/locustio/locust/tree/master/examples/custom_shape>`_ bao gồm:

- Tạo hình dạng sóng kép
- Các giai đoạn dựa trên thời gian như K6
- Mẫu tải bước như Visual Studio

Một phương pháp khác có thể hữu ích cho các hình dạng tải tùy chỉnh của bạn: `get_current_user_count()`, trả về tổng số người dùng hoạt động. Phương pháp này có thể được sử dụng để ngăn chặn tiến tới các bước tiếp theo cho đến khi số lượng người dùng mong muốn đã đạt được. Điều này đặc biệt hữu ích nếu quá trình khởi tạo cho mỗi người dùng chậm hoặc không ổn định trong việc mất bao lâu. Nếu đây là trường hợp sử dụng của bạn, hãy xem `ví dụ trên github <https://github.com/locustio/locust/tree/master/examples/custom_shape/wait_user_count.py>`_.

Lưu ý rằng nếu bạn muốn xác định hình dạng cơ bản tùy chỉnh của riêng mình, bạn cần xác định thuộc tính `abstract` thành `True` để tránh nó được chọn làm Shape khi được nhập:

.. code-block:: python

    class MyBaseShape(LoadTestShape):
        abstract = True
        
        def tick(self):
            # Một cái gì đó có thể tái sử dụng nhưng cần kế thừa
            return None


Kết hợp người dùng với các hình dạng tải khác nhau
--------------------------------------------

Nếu bạn sử dụng giao diện Web, bạn có thể thêm tham số :ref:`---class-picker <class-picker>` để chọn hình dạng nào sẽ được sử dụng. Nhưng thường linh hoạt hơn khi bạn định nghĩa các định nghĩa Người dùng của mình trong một tệp và hình dạng LoadTestShape trong một tệp riêng. Ví dụ, nếu bạn có một lớp hình dạng tải cao/thấp được xác định trong low_load.py và high_load.py tương ứng:

.. code-block:: console

    $ locust -f locustfile.py,low_load.py

    $ locust -f locustfile.py,high_load.py

Hạn chế loại người dùng nào sẽ spawn trong mỗi tick
--------------------------------------------------

Thêm phần tử ``user_classes`` vào giá trị trả về sẽ cho bạn kiểm soát chi tiết hơn:

.. code-block:: python

    class StagesShapeWithCustomUsers(LoadTestShape):

        stages = [
            {"duration": 10, "users": 10, "spawn_rate": 10, "user_classes": [UserA]},
            {"duration": 30, "users": 50, "spawn_rate": 10, "user_classes": [UserA, UserB]},
            {"duration": 60, "users": 100, "spawn_rate": 10, "user_classes": [UserB]},
            {"duration": 120, "users": 100, "spawn_rate": 10, "user_classes": [UserA,UserB]},
        ]

        def tick(self):
            run_time = self.get_run_time()

            for stage in self.stages:
                if run_time < stage["duration"]:
                    try:
                        tick_data = (stage["users"], stage["spawn_rate"], stage["user_classes"])
                    except:
                        tick_data = (stage["users"], stage["spawn_rate"])
                    return tick_data

            return None

Hình dạng này sẽ tạo ra trong 10 giây đầu tiên 10 người dùng của ``UserA``. Trong 20 giây tiếp theo 40 người dùng của loại ``UserA/UserB`` và điều này tiếp tục cho đến khi các giai đoạn kết thúc.


.. _use-common-options:

Sử dụng các tùy chọn chung trong các hình dạng tùy chỉnh
---------------------------------------

Khi sử dụng hình dạng, các tùy chọn *Users*, *Spawn Rate* và *Run Time* sẽ bị ẩn khỏi giao diện người dùng, và nếu bạn xác định chúng trên dòng lệnh Locust sẽ ghi lại một cảnh báo. Điều này là vì những tùy chọn đó không áp dụng trực tiếp cho các hình dạng, và việc xác định chúng có thể là một lỗi.

Nếu bạn thực sự muốn kết hợp một hình dạng với các tùy chọn này, hãy đặt thuộc tính ``use_common_options`` và truy cập chúng từ ``self.runner.environment.parsed_options``:

.. code-block:: python

    class MyCustomShape(LoadTestShape):
        use_common_options = True
        
        def tick(self):
            run_time = self.get_run_time()
            if run_time < self.runner.environment.parsed_options.run_time:
                # Số lượng người dùng làm tròn đến hàng trăm gần nhất, giống như trong ví dụ trước
                user_count = round(run_time, -2)
                return (user_count, self.runner.environment.parsed_options.spawn_rate)
                
            return None
