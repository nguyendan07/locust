.. _running-without-web-ui:

=================================
Chạy mà không có giao diện web (Running without the web UI)
=================================

Bạn có thể chạy Locust mà không cần giao diện web bằng cách sử dụng cờ ``--headless`` cùng với ``-u/--users`` và ``-r/--spawn-rate``:

.. code-block:: console
    :substitutions:

    $ locust -f locust_files/my_locust_file.py --headless -u 100 -r 5
    [2021-07-24 10:41:10,947] .../INFO/locust.main: No run time limit set, use CTRL+C to interrupt.
    [2021-07-24 10:41:10,947] .../INFO/locust.main: Starting Locust |version|
    [2021-07-24 10:41:10,949] .../INFO/locust.runners: Ramping to 100 users using a 5.00 spawn rate
    Name              # reqs      # fails  |     Avg     Min     Max  Median  |   req/s failures/s
    ----------------------------------------------------------------------------------------------
    GET /hello             1     0(0.00%)  |     115     115     115     115  |    0.00    0.00
    GET /world             1     0(0.00%)  |     119     119     119     119  |    0.00    0.00
    ----------------------------------------------------------------------------------------------
    Aggregated             2     0(0.00%)  |     117     115     119     117  |    0.00    0.00
    (...)
    [2021-07-24 10:44:42,484] .../INFO/locust.runners: All users spawned: {"HelloWorldUser": 100} (100 total users)
    (...)

Ngay cả trong chế độ headless, bạn vẫn có thể thay đổi số lượng người dùng trong khi bài kiểm tra đang chạy. Nhấn ``w`` để thêm 1 người dùng hoặc ``W`` để thêm 10. Nhấn ``s`` để xóa 1 hoặc ``S`` để xóa 10.

Thiết lập thời gian chạy cho bài kiểm tra
---------------------------------

Để chỉ định thời gian chạy cho một bài kiểm tra, sử dụng ``-t/--run-time``:

.. code-block:: console

    $ locust --headless -u 100 --run-time 1h30m
    $ locust --headless -u 100 --run-time 60 # đơn vị mặc định là giây

Locust sẽ dừng lại khi hết thời gian. Thời gian được tính từ lúc bắt đầu bài kiểm tra (không phải từ khi hoàn thành việc tăng dần).


Cho phép các nhiệm vụ hoàn thành vòng lặp của mình khi tắt
-------------------------------------------------

Mặc định, Locust sẽ dừng các nhiệm vụ của bạn ngay lập tức (mà không cần chờ các yêu cầu kết thúc).
Để cho các nhiệm vụ đang chạy một thời gian để hoàn thành vòng lặp của họ, sử dụng ``-s/--stop-timeout``:

.. code-block:: console

    $ locust --headless --run-time 1h30m --stop-timeout 10s

.. _running-distributed-without-web-ui:


Chạy Locust phân tán mà không có giao diện web
---------------------------------------------

Nếu bạn muốn :ref:`chạy Locust phân tán <running-distributed>` mà không có giao diện web,
bạn nên chỉ định tùy chọn ``--expect-workers`` khi bắt đầu nút master, để chỉ định
số lượng nút worker được kỳ vọng sẽ kết nối. Sau đó, nó sẽ chờ cho đến khi có đủ nhiều nút worker
đã kết nối trước khi bắt đầu bài kiểm tra.


Kiểm soát mã thoát của quá trình Locust
-----------------------------------------------

Mặc định quá trình locust sẽ cung cấp mã thoát là 1 nếu có bất kỳ mẫu nào thất bại
(sử dụng ``--exit-code-on-error`` để thay đổi hành vi đó).

Bạn cũng có thể kiểm soát mã thoát một cách thủ công trong các tập lệnh kiểm tra của bạn bằng cách thiết lập :py:attr:`process_exit_code <locust.env.Environment.process_exit_code>` của
:py:class:`Environment <locust.env.Environment>` instance. Điều này đặc biệt hữu ích khi chạy Locust như một bài kiểm tra tự động/lên lịch, ví dụ như là một phần của pipeline CI.

Dưới đây là một ví dụ sẽ thiết lập mã thoát thành không phải không nếu bất kỳ điều kiện sau đây được đáp ứng:

* Hơn 1% các yêu cầu thất bại
* Thời gian phản hồi trung bình dài hơn 200 ms
* Phần trăm 95 cho thời gian phản hồi lớn hơn 800 ms

.. code-block:: python

    import logging
    from locust import events
    
    @events.quitting.add_listener
    def _(environment, **kw):
        if environment.stats.total.fail_ratio > 0.01:
            logging.error("Test failed due to failure ratio > 1%")
            environment.process_exit_code = 1
        elif environment.stats.total.avg_response_time > 200:
            logging.error("Test failed due to average response time ratio > 200 ms")
            environment.process_exit_code = 1
        elif environment.stats.total.get_response_time_percentile(0.95) > 800:
            logging.error("Test failed due to 95th percentile response time > 800 ms")
            environment.process_exit_code = 1
        else:
            environment.process_exit_code = 0

Note that this code could go into the locustfile.py or in any other file that is imported in the locustfile.
