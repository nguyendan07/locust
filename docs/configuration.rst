.. _configuration:

=============
Configuration
=============


Command Line Options
====================

Locust được cấu hình chủ yếu thông qua các đối số dòng lệnh (command line arguments).

.. code-block:: console

    $ locust --help

.. literalinclude:: cli-help-output.txt
    :language: console



.. _environment-variables:

Biến môi trường (Environment Variables)
=====================

Các tùy chọn cũng có thể được đặt thông qua các biến môi trường. Thông thường chúng giống như đối số dòng lệnh
nhưng viết hoa và thêm tiền tố ``LOCUST_``:

Trên Linux/macOS:

.. code-block:: console

    $ LOCUST_LOCUSTFILE=custom_locustfile.py locust

Trên Windows:

.. code-block:: console

    > set LOCUST_LOCUSTFILE=custom_locustfile.py
    > locust

.. _configuration-file:

Tệp cấu hình (Configuration File)
==================

Tùy chọn cũng có thể được đặt trong một tệp cấu hình trong định dạng
`config hoặc TOML <https://github.com/bw2/ConfigArgParse#config-file-syntax>`_.

Locust sẽ tìm kiếm ``~/.locust.conf``, ``./locust.conf`` và ``./pyproject.toml`` theo mặc định.
Bạn có thể chỉ định một tệp bổ sung bằng cờ ``--config``.

.. code-block:: console

    $ locust --config custom_config.conf

Dưới đây là một ví dụ nhanh về các tệp cấu hình được hỗ trợ bởi Locust:

locust.conf
--------------

.. code-block:: ini

    locustfile = locust_files/my_locust_file.py
    headless = true
    master = true
    expect-workers = 5
    host = https://target-system
    users = 100
    spawn-rate = 10
    run-time = 10m
    tags = [Critical, Normal]

pyproject.toml
--------------

Khi sử dụng tệp TOML, các tùy chọn cấu hình nên được xác định trong phần ``[tool.locust]``.

.. code-block:: toml

    [tool.locust]
    locustfile = "locust_files/my_locust_file.py"
    headless = true
    master = true
    expect-workers = 5
    host = "https://target-system"
    users = 100
    spawn-rate = 10
    run-time = "10m"
    tags = ["Critical", "Normal"]

.. note::

    Các giá trị cấu hình được đọc (và ghi đè) theo thứ tự sau:
    
    .. code-block:: console
        
       ~/.locust.conf -> ./locust.conf -> ./pyproject.toml -> (file specified using --conf) -> env vars -> cmd args


Tất cả các tùy chọn cấu hình có sẵn
===================================

Dưới đây là một bảng của tất cả các tùy chọn cấu hình có sẵn, và các khóa tương ứng trong môi trường và tệp cấu hình:

.. include:: config-options.rst

Chạy mà không có giao diện web
==========================

Xem :ref:`running-without-web-ui`

Sử dụng nhiều tệp Locust cùng một lúc
==================================

``-f/--locustfile`` chấp nhận nhiều tệp locust, được phân tách bằng dấu phẩy.

Ví dụ:

Với cấu trúc tệp sau:

.. code-block::

    ├── locustfiles/
    │   ├── locustfile1.py
    │   ├── locustfile2.py
    │   └── more_files/
    │       ├── locustfile3.py
    │       ├── _ignoreme.py

.. code-block:: console

    $ locust -f locustfiles/locustfile1.py,locustfiles/locustfile2.py,locustfiles/more_files/locustfile3.py

Locust sẽ sử dụng ``locustfile1.py``, ``locustfile2.py`` và ``more_files/locustfile3.py``

Ngoài ra, ``-f/--locustfile`` chấp nhận thư mục là một tùy chọn. Locust sẽ tìm kiếm đệ quy các thư mục được chỉ định cho các tệp ``*.py``, bỏ qua các tệp bắt đầu bằng "_".

Ví dụ:

.. code-block:: console

    $ locust -f locustfiles

Locust sẽ sử dụng ``locustfile1.py``, ``locustfile2.py`` và ``more_files/locustfile3.py``

Bạn cũng có thể sử dụng ``-f/--locustfile`` cho các URL web. Điều này sẽ tải tệp và sử dụng nó như bất kỳ tệp locust nào.

Ví dụ:

.. code-block:: console

    $ locust -f https://raw.githubusercontent.com/locustio/locust/master/examples/basic.py


.. _class-picker:

Chọn lớp người dùng, hình dạng và nhiệm vụ từ giao diện người dùng
===============================================

Bạn có thể chọn lớp Shape và lớp User nào để chạy trong WebUI khi chạy locust với cờ ``--class-picker``.
Không có lựa chọn nào sẽ sử dụng tất cả các lớp User có sẵn.

Ví dụ, với cấu trúc tệp như sau:

.. code-block::

    ├── src/
    │   ├── some_file.py
    ├── locustfiles/
    │   ├── locustfile1.py
    │   ├── locustfile2.py
    │   └── more_files/
    │       ├── locustfile3.py
    │       ├── _ignoreme.py
    │   └── shape_classes/
    │       ├── DoubleWaveShape.py
    │       ├── StagesShape.py


.. code-block:: console

    $ locust -f locustfiles --class-picker

Giao diện người dùng web sẽ hiển thị:

.. image:: images/userclass_picker_example.png

Công cụ chọn lớp bổ sung cho phép vô hiệu hóa các tác vụ User, thay đổi trọng lượng hoặc số lượng cố định, và cấu hình máy chủ.

Thậm chí còn có thể thêm các thuộc tính tùy chỉnh mà bạn muốn được cấu hình cho mỗi User. Đơn giản thêm một phương thức ``json`` vào User của bạn:

.. code-block:: python

    class Example(HttpUser):
        @task
        def example_task(self):
            self.client.get(f"/example/{self.some_custom_arg}")

        @classmethod
        def json(self):
            return {
                "host": self.host,
                "some_custom_arg": "example"
            }

Cấu hình Users từ dòng lệnh
=================================

Bạn có thể cập nhật các thuộc tính lớp User từ dòng lệnh, sử dụng đối số ``--config-users``:

.. code-block:: console

    $ locust --config-users '{"user_class_name": "Example", "fixed_count": 1, "some_custom_attribute": false}'

Để cấu hình nhiều người dùng, bạn truyền nhiều đối số cho ``--config-users``, hoặc sử dụng một Mảng JSON. Bạn cũng có thể truyền đường dẫn đến tệp JSON.

.. code-block:: console

    $ locust --config-users '{"user_class_name": "Example", "fixed_count": 1}' '{"user_class_name": "ExampleTwo", "fixed_count": 2}'
    $ locust --config-users '[{"user_class_name": "Example", "fixed_count": 1}, {"user_class_name": "ExampleTwo", "fixed_count": 2}]'
    $ locust --config-users my_user_config.json

Khi sử dụng cách này để cấu hình người dùng của bạn, bạn có thể đặt bất kỳ thuộc tính nào.

.. note::

    ``--config-users`` là một tính năng hơi thử nghiệm và định dạng json có thể thay đổi ngay cả giữa các bản Locust nhỏ.

Đối số tùy chỉnh (Custom arguments)
================

Xem :ref:`custom-arguments`

Tùy chỉnh cài đặt thống kê
====================================

Cấu hình mặc định cho thống kê Locust được đặt trong các hằng số của tệp stats.py.
Nó có thể được điều chỉnh theo yêu cầu cụ thể bằng cách ghi đè các giá trị này.
Để làm điều này, nhập mô-đun locust.stats và ghi đè các cài đặt cần thiết:

.. code-block:: python

    import locust.stats
    locust.stats.CONSOLE_STATS_INTERVAL_SEC = 15

Điều này có thể được thực hiện trực tiếp trong tệp Locust hoặc được trích xuất thành tệp riêng cho việc sử dụng chung bởi tất cả các tệp Locust.

Danh sách các tham số thống kê có thể được sửa đổi là:

+-------------------------------------------+-------------------------------------------------------------------------------------------------+
| Parameter name                            | Purpose                                                                                         |
+-------------------------------------------+-------------------------------------------------------------------------------------------------+
| STATS_NAME_WIDTH                          | Độ rộng của cột cho tên yêu cầu trong đầu ra bảng điều khiển                                    |
+-------------------------------------------+-------------------------------------------------------------------------------------------------+
| STATS_TYPE_WIDTH                          | Độ rộng của cột cho loại yêu cầu trong đầu ra bảng điều khiển                                   |
+-------------------------------------------+-------------------------------------------------------------------------------------------------+
| CSV_STATS_INTERVAL_SEC                    | Khoảng thời gian để tệp CSV được ghi thường xuyên như thế nào nếu tùy chọn này được cấu hình    |
+-------------------------------------------+-------------------------------------------------------------------------------------------------+
| CONSOLE_STATS_INTERVAL_SEC                | Khoảng thời gian để kết quả được ghi vào bảng điều khiển thường xuyên như thế nào               |
+-------------------------------------------+-------------------------------------------------------------------------------------------------+
| CURRENT_RESPONSE_TIME_PERCENTILE_WINDOW   | Window size/resolution - in seconds - when calculating the current response                     |
|                                           | time percentile                                                                                 |
+-------------------------------------------+-------------------------------------------------------------------------------------------------+
| PERCENTILES_TO_REPORT                     | Danh sách các phần trăm thời gian phản hồi được tính toán & báo cáo                             |
+-------------------------------------------+-------------------------------------------------------------------------------------------------+
| PERCENTILES_TO_CHART                      | Danh sách các phần trăm thời gian phản hồi trong màn hình biểu đồ cho UI                        |
+-------------------------------------------+-------------------------------------------------------------------------------------------------+
| PERCENTILES_TO_STATISTICS                 | Danh sách các phần trăm thời gian phản hồi trong màn hình thống kê cho UI                       |
+-------------------------------------------+-------------------------------------------------------------------------------------------------+

Tùy chỉnh các biến tĩnh bổ sung (Customization of additional static variables)
============================================

Bảng này liệt kê các hằng số được đặt trong locust và có thể bị ghi đè.

+-------------------------------------------+------------------------------------------------------------------------------------------------+
| Parameter name                            | Purpose                                                                                        |
+-------------------------------------------+------------------------------------------------------------------------------------------------+
| locust.runners.WORKER_LOG_REPORT_INTERVAL | Khoảng thời gian để nhật ký công nhân được báo cáo cho máy chủ thường xuyên như thế nào.       |
|                                           | Có thể vô hiệu hóa bằng cách đặt thành một số âm                                               |
+-------------------------------------------+------------------------------------------------------------------------------------------------+
