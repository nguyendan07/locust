.. _logging:

=======
Logging
=======

Locust sử sụng `logging framework <https://docs.python.org/3/library/logging.html>`_ của Python để xử lý logging.

Cấu hình logging mặc định mà Locust áp dụng, ghi các thông báo log trực tiếp vào stderr. ``--loglevel`` và ``--logfile``
có thể được sử dụng để thay đổi mức độ chi tiết và/hoặc làm cho log được ghi vào một file thay thế.

Cấu hình logging mặc định cài đặt các handlers cho logger ``root`` cũng như các logger ``locust.*``,
vì vậy việc sử dụng logger root trong các script test của bạn sẽ đưa thông báo vào file log nếu ``--logfile`` được sử dụng:

.. code-block:: python

    import logging
    logging.info("this log message will go wherever the other locust log messages go")

Cũng có thể kiểm soát toàn bộ cấu hình logging trong các script test của bạn bằng cách sử dụng tùy chọn ``--skip-log-setup``.
Sau đó, bạn sẽ phải `cấu hình logging <https://docs.python.org/3/library/logging.config.html>`_ của mình.


Options
=======

``--skip-log-setup``
--------------------

Vô hiệu hóa cấu hình logging của Locust. Thay vào đó, cấu hình được cung cấp bởi test Locust hoặc mặc định của Python.


``--loglevel``
--------------

Chọn giữa DEBUG/INFO/WARNING/ERROR/CRITICAL. Mặc định là INFO. Phiên bản viết tắt là ``-L``.


``--logfile``
-------------

Đường dẫn đến file log. Nếu không được thiết lập, log sẽ được ghi vào stdout/stderr.


Locust loggers
==============

Dưới đây là một bảng về các loggers được sử dụng trong Locust (để tham khảo khi cấu hình các thiết lập logging thủ công):

+------------------------+--------------------------------------------------------------------------------------+
| Logger name            | Mục đích                                                                             |
+------------------------+--------------------------------------------------------------------------------------+
| locust                 | Namespace locust được sử dụng cho tất cả các loggers như ``locust.main``,            |
|                        | ``locust.runners``, v.v.                                                             |
+------------------------+--------------------------------------------------------------------------------------+
| locust.stats_logger    | Logger này được sử dụng để định kỳ in các stats hiện tại ra console. Stats không     |
|                        | được ghi vào file log khi ``--logfile`` được sử dụng theo mặc định.                  |
+------------------------+--------------------------------------------------------------------------------------+
