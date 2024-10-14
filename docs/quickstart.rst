.. _quickstart:

===============
Hướng dẫn nhanh
===============
Một bài kiểm tra Locust về cơ bản chỉ là một chương trình Python thực hiện các yêu cầu đến hệ thống bạn muốn kiểm tra. Điều này khiến nó rất linh hoạt và đặc biệt phù hợp để thực hiện các luồng người dùng phức tạp. Nhưng nó cũng có thể thực hiện các bài kiểm tra đơn giản, vì vậy hãy bắt đầu với điều đó:

.. code-block:: python

    from locust import HttpUser, task

    class HelloWorldUser(HttpUser):
        @task
        def hello_world(self):
            self.client.get("/hello")
            self.client.get("/world")

Đoạn mã trên sẽ thực hiện một yêu cầu HTTP đến ``/hello``, sau đó đến ``/world``, và sau đó lặp lại. Để biết thêm thông tin và một ví dụ thực tế hơn, xem :ref:`writing-a-locustfile`.

Để chạy bài kiểm tra này, hãy thay đổi ``/hello`` và ``/world`` thành một số đường dẫn thực tế trên trang web/dịch vụ bạn muốn kiểm tra, đặt mã vào một tệp có tên ``locustfile.py`` trong thư mục hiện tại và sau đó chạy ``locust``:

.. code-block:: console
    :substitutions:

    $ locust
    [2021-07-24 09:58:46,215] .../INFO/locust.main: Starting web interface at http://0.0.0.0:8089
    [2021-07-24 09:58:46,285] .../INFO/locust.main: Starting Locust |version|

Giao diện web của Locust
======================

Mở trình duyệt và truy cập vào ``http://localhost:8089``

.. image:: images/webui-splash-screenshot.png

| Cung cấp tên máy chủ của máy chủ của bạn và thử nó!


Các ảnh chụp màn hình sau đây cho thấy nó có thể trông như thế nào khi chạy thử nghiệm này bằng 50 người dùng đồng thời, với tốc độ tăng dần là 1 người dùng/giây

.. image:: images/webui-running-statistics.png

| Dưới tab *Charts*, bạn sẽ tìm thấy các thứ như số yêu cầu mỗi giây (RPS), thời gian phản hồi và số người dùng đang chạy:

.. image:: images/bottlenecked_server.png

.. note::

    Việc giải thích kết quả kiểm tra hiệu suất khá phức tạp (và chủ yếu nằm ngoài phạm vi của hướng dẫn này), nhưng nếu biểu đồ của bạn bắt đầu trông như thế này, dịch vụ/hệ thống mục tiêu không thể xử lý tải và bạn đã tìm thấy một chướng ngại vật.

    Khi chúng ta đạt đến khoảng 20 người dùng, thời gian phản hồi bắt đầu tăng nhanh đến mức mà mặc dù Locust vẫn tạo ra thêm người dùng, số yêu cầu mỗi giây không còn tăng nữa. Dịch vụ mục tiêu bị "quá tải" hoặc "bão hòa".

    Nếu thời gian phản hồi của bạn *không* tăng lên thì thêm nhiều người dùng hơn cho đến khi bạn tìm thấy điểm phá vỡ của dịch vụ, hoặc chúc mừng vì dịch vụ của bạn đã đủ hiệu quả cho tải dự kiến của bạn.

    Nếu bạn gặp khó khăn trong việc tạo ra đủ tải để bão hòa hệ thống của mình, hãy xem :ref:`increaserr`.

Sử dụng dòng lệnh trực tiếp / không có giao diện
====================================

Việc sử dụng giao diện web Locust là hoàn toàn tùy chọn. Bạn có thể cung cấp các tham số tải trên dòng lệnh và nhận báo cáo về kết quả dưới dạng văn bản:

.. code-block:: console
    :substitutions:

    $ locust --headless --users 10 --spawn-rate 1 -H http://your-server.com
    [2021-07-24 10:41:10,947] .../INFO/locust.main: No run time limit set, use CTRL+C to interrupt.
    [2021-07-24 10:41:10,947] .../INFO/locust.main: Starting Locust |version|
    [2021-07-24 10:41:10,949] .../INFO/locust.runners: Ramping to 10 users using a 1.00 spawn rate
    Name              # reqs      # fails  |     Avg     Min     Max  Median  |   req/s failures/s
    ----------------------------------------------------------------------------------------------
    GET /hello             1     0(0.00%)  |     115     115     115     115  |    0.00    0.00
    GET /world             1     0(0.00%)  |     119     119     119     119  |    0.00    0.00
    ----------------------------------------------------------------------------------------------
    Aggregated             2     0(0.00%)  |     117     115     119     117  |    0.00    0.00
    (...)
    [2021-07-24 10:44:42,484] .../INFO/locust.runners: All users spawned: {"HelloWorldUser": 10} (10 total users)
    (...)


Xem :ref:`running-without-web-ui` để biết thêm chi tiết.

Thêm tùy chọn
============

Để chạy Locust phân tán trên nhiều quy trình Python hoặc máy, bạn bắt đầu một quy trình Locust master duy nhất với tham số dòng lệnh ``--master``, sau đó bất kỳ số lượng quy trình Locust worker nào sử dụng tham số dòng lệnh ``--worker``. Xem :ref:`running-distributed` để biết thêm thông tin.

Để xem tất cả các tùy chọn có sẵn, hãy gõ: ``locust --help`` hoặc kiểm tra :ref:`configuration`.

Bước tiếp theo
==========

Now, let's have a more in-depth look at locustfiles and what they can do: :ref:`writing-a-locustfile`.
