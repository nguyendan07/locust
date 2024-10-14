.. _running-in-docker:

=================
Chạy trong Docker (Running in Docker)
=================

Hình ảnh Docker chính thức nằm ở `locustio/locust <https://hub.docker.com/r/locustio/locust>`_.

Sử dụng nó như sau (giả sử rằng ``locustfile.py`` tồn tại trong thư mục làm việc hiện tại)::

    docker run -p 8089:8089 -v $PWD:/mnt/locust locustio/locust -f /mnt/locust/locustfile.py

Trên Windows, lệnh này đôi khi sẽ găp lỗi. Người dùng Windows nên thử sử dụng cái này thay vào đó::

    docker run -p 8089:8089 --mount type=bind,source=$pwd,target=/mnt/locust locustio/locust -f /mnt/locust/locustfile.py

Docker Compose
==============

Dưới đây là một tệp Docker Compose mẫu có thể được sử dụng để bắt đầu cả nút master và nút worker:

.. literalinclude:: ../examples/docker-compose/docker-compose.yml
    :language: yaml

Cấu hình compose trên có thể được sử dụng để bắt đầu cả nút master và 4 nút worker bằng lệnh sau::
    
    docker-compose up --scale worker=4

Sử dụng hình ảnh docker như một hình ảnh cơ sở (Use docker image as a base image)
================================

Rất phổ biến khi có các tập lệnh kiểm thử của bản thứ ba. Trong những trường hợp đó, bạn có thể sử dụng hình ảnh docker chính thức của Locust như một hình ảnh cơ sở::

    FROM locustio/locust
    RUN pip3 install some-python-package

Chạy Locust bằng Kubernetes
===============================

Xem :ref:`Helm charts for Locust <helm>`