.. _installation:

Cài đặt
============

.. note::

    Nếu gặp sự cố, hãy tham khảo mục `Xử lý sự cố cài đặt`_.

0. `Cài đặt Python <https://docs.python-guide.org/starting/installation/>`_ (nếu chưa có)

1. Cài đặt Locust

.. code-block:: console

    $ pip3 install locust

2. Kiểm tra cài đặt

.. code-block:: console
    :substitutions:

    $ locust -V
    locust |version| from /usr/local/lib/python3.12/site-packages/locust (Python 3.12.5)


Sử dụng uvx (lựa chọn thay thế)
-----------------------

0. `Cài đặt uv <https://github.com/astral-sh/uv?tab=readme-ov-file#installation>`_

1. Cài đặt và chạy Locust trong môi trường tạm thời

.. code-block:: console
    :substitutions:

    $ uvx locust -V
    locust |version| from /.../uv/.../locust (Python 3.12.5)

Xong!
-----

Bây giờ bạn có thể :ref:`tạo và chạy bài kiểm tra đầu tiên <quickstart>` của mình.

-----------------


Bản dựng trước khi phát hành
------------------

Nếu bạn cần phiên bản mới nhất của Locust và không thể đợi đến bản phát hành tiếp theo, bạn có thể cài đặt bản dựng dev như sau:

.. code-block:: console

    $ pip3 install -U --pre locust

Bản dựng trước khi phát hành được công bố mỗi khi nhánh/PR được hợp nhất vào bản chính.

Cài đặt để phát triển
-----------------------

Nếu bạn muốn sửa đổi Locust hoặc đóng góp cho dự án, hãy xem :ref:`developing-locust`.

Xử lý sự cố cài đặt
----------------------------


.. contents:: Một số giải pháp cho các sự cố cài đặt thông thường
    :depth: 1
    :local:
    :backlinks: none


psutil/\_psutil_common.c:9:10: fatal error: Python.h: No such file or directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`Trả lời trong chuỗi Stackoverflow 63440765 <https://stackoverflow.com/questions/63440765/locust-installation-error-using-pip3-error-command-errored-out-with-exit-statu>`_

ERROR: Failed building wheel for xxx
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mặc dù Locust là một gói Python thuần túy, nó có một số phụ thuộc (ví dụ: gevent và geventhttpclient) được biên dịch từ mã C. Hầu hết các nền tảng thông dụng đều có các gói nhị phân trên PyPI, nhưng đôi khi có phiên bản mới không có hoặc bạn đang chạy trên một nền tảng lạ. Bạn có hai lựa chọn:

- (trên macOS) Cài đặt xcode: ``xcode-select --install``
- Sử dụng ``pip install --prefer-binary locust`` để chọn phiên bản được biên dịch sẵn của các gói ngay cả khi có phiên bản mới hơn dưới dạng source.
- Thử tìm kiếm thông báo lỗi trên Google cho gói cụ thể bị lỗi (không phải Locust), đảm bảo bạn đã cài đặt các công cụ build thích hợp, v.v.

Windows
~~~~~~~

`Trả lời trong chuỗi Stackoverflow 61592069 <https://stackoverflow.com/questions/61592069/locust-is-not-installing-on-my-windows-10-for-load-testing>`_

Cài đặt thành công, nhưng lệnh ``locust`` không được tìm thấy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Khi chạy pip, bạn có nhận được cảnh báo cho biết ``The script locust is installed in '...' which is not on PATH`` không?

Hãy thêm thư mục đó vào biến môi PATH của bạn.

Tăng giới hạn số lượng tệp mở tối đa
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mỗi kết nối Người dùng/HTTP từ Locust mở một tệp mới (về mặt kỹ thuật là một trình mô tả tệp). Nhiều hệ điều hành theo mặc định đặt giới hạn thấp cho số lượng tệp tối đa có thể được mở cùng một lúc (điều này thường được gọi là "giới hạn số lượng tệp mở"). Locust sẽ cố gắng điều chỉnh điều này tự động cho bạn, nhưng trong nhiều trường hợp hệ điều hành của bạn sẽ không cho phép điều này (trong trường hợp đó bạn sẽ nhận được cảnh báo trong log). Thay vào đó, bạn sẽ phải làm điều này thủ công.

Cách thực hiện điều này phụ thuộc vào hệ điều hành của bạn, nhưng bạn có thể tìm thấy một số thông tin hữu ích tại đây:
https://www.tecmint.com/increase-set-open-file-limits-in-linux/ và ví dụ thực tế
https://www.ibm.com/support/knowledgecenter/SS8NLW_11.0.2/com.ibm.discovery.es.in.doc/iiysiulimits.html

Đối với các hệ thống dựa trên systemd (ví dụ: Debian/Ubuntu) các giới hạn khác nhau được sử dụng cho các phiên đăng nhập đồ họa. Xem
https://unix.stackexchange.com/a/443467 để biết các cài đặt bổ sung.
