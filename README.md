# Locust

[![PyPI](https://img.shields.io/pypi/v/locust.svg)](https://pypi.org/project/locust/)<!--![Python Version from PEP 621 TOML](https://img.shields.io/python/required-version-toml?tomlFilePath=https%3A%2F%2Fraw.githubusercontent.com%2Flocustio%2Flocust%2Fmaster%2Fpyproject.toml)-->[![Downloads](https://pepy.tech/badge/locust/week)](https://pepy.tech/project/locust)
[![Build Status](https://github.com/locustio/locust/workflows/Tests/badge.svg)](https://github.com/locustio/locust/actions?query=workflow%3ATests)
[![GitHub contributors](https://img.shields.io/github/contributors/locustio/locust.svg)](https://github.com/locustio/locust/graphs/contributors)
[![Support Ukraine Badge](https://bit.ly/support-ukraine-now)](https://github.com/support-ukraine/support-ukraine)

Locust là một công cụ kiểm tra hiệu suất/tải mã nguồn mở cho HTTP và các giao thức khác. Cách tiếp cận thân thiện với nhà phát triển của nó cho phép bạn định nghĩa các bài kiểm tra của mình bằng mã Python thông thường.

Các bài kiểm tra Locust có thể được chạy từ dòng lệnh hoặc sử dụng giao diện web của nó. Tốc độ xử lý, thời gian phản hồi và lỗi có thể được xem theo thời gian thực và/hoặc xuất ra để phân tích sau này.

Bạn có thể nhập (import) các thư viện Python thông thường vào các bài kiểm tra của mình, và với kiến trúc có thể cắm thêm của Locust, nó có thể mở rộng vô hạn. Không giống như khi sử dụng hầu hết các công cụ khác, thiết kế bài kiểm tra của bạn sẽ không bao giờ bị giới hạn bởi giao diện người dùng hoặc ngôn ngữ miền cụ thể.

Để bắt đầu ngay lập tức, hãy truy cập vào [tài liệu](http://docs.locust.io/en/stable/installation.html).

## Features

#### Viết kịch bản kiểm tra người dùng bằng Python thông thường

Nếu bạn muốn người dùng của mình lặp lại, thực hiện một số hành vi có điều kiện hoặc thực hiện một số tính toán, bạn chỉ cần sử dụng các cấu trúc lập trình thông thường do Python cung cấp. Locust chạy mỗi người dùng bên trong greenlet của riêng nó (một quy trình nhẹ/coroutine). Điều này cho phép bạn viết các bài kiểm tra của mình như mã Python bình thường (blocking) thay vì phải sử dụng các callback hoặc một số cơ chế khác. Vì các kịch bản của bạn là "chỉ là python" nên bạn có thể sử dụng IDE thông thường của mình và kiểm soát phiên bản các bài kiểm tra của mình như mã thông thường (trái ngược với một số công cụ khác sử dụng định dạng XML hoặc nhị phân).

```python
from locust import HttpUser, task, between

class QuickstartUser(HttpUser):
    wait_time = between(1, 2)

    def on_start(self):
        self.client.post("/login", json={"username":"foo", "password":"bar"})

    @task
    def hello_world(self):
        self.client.get("/hello")
        self.client.get("/world")

    @task(3)
    def view_item(self):
        for item_id in range(10):
            self.client.get(f"/item?id={item_id}", name="/item")
```

#### Phân tán & Mở rộng - hỗ trợ hàng trăm nghìn người dùng

Locust giúp bạn dễ dàng chạy các bài kiểm tra tải phân tán trên nhiều máy. Nó dựa trên sự kiện (sử dụng [gevent](http://www.gevent.org/)), cho phép một quy trình duy nhất xử lý hàng nghìn người dùng đồng thời. Mặc dù có thể có các công cụ khác có khả năng thực hiện nhiều yêu cầu mỗi giây hơn trên một phần cứng nhất định, nhưng chi phí thấp của mỗi người dùng Locust khiến nó rất phù hợp để kiểm tra các tải công việc đồng thời cao.

#### Web-based UI

Locust có giao diện web thân thiện với người dùng, hiển thị tiến trình kiểm tra của bạn theo thời gian thực. Bạn thậm chí có thể thay đổi tải trong khi kiểm tra đang chạy. Nó cũng có thể được chạy mà không cần giao diện người dùng, giúp dễ dàng sử dụng cho kiểm tra CI/CD.

<img src="docs/images/bottlenecked_server.png" alt="Locust UI charts" height="100" width="200"/> <img src="docs/images/webui-running-statistics.png" alt="Locust UI stats" height="100" width="200"/> <img src="docs/images/locust_workers.png" alt="Locust UI workers" height="100" width="200"/> <img src="docs/images/webui-splash-screenshot.png" alt="Locust UI start test" height="100" width="200"/>

#### Có thể kiểm tra bất kỳ hệ thống nào
Mặc dù Locust chủ yếu làm việc với các trang web/dịch vụ, nhưng nó có thể được sử dụng để kiểm tra hầu như bất kỳ hệ thống hoặc giao thức nào. Chỉ cần [viết một client](https://docs.locust.io/en/latest/testing-other-systems.html#testing-other-systems) cho những gì bạn muốn kiểm tra, hoặc [khám phá một số client được tạo bởi cộng đồng](https://github.com/SvenskaSpel/locust-plugins#users).

## Hackable

Mã nguồn của Locust được giữ nhỏ gọn và không giải quyết mọi thứ ngay từ đầu. Thay vào đó, chúng tôi cố gắng làm cho nó dễ dàng thích ứng với bất kỳ tình huống nào bạn có thể gặp phải, bằng cách sử dụng mã Python thông thường. Không có gì ngăn cản bạn:

* [Send real time reporting data to TimescaleDB and visualize it in Grafana](https://github.com/SvenskaSpel/locust-plugins/blob/master/locust_plugins/dashboards/README.md)
* [Wrap calls to handle the peculiarities of your REST API](https://github.com/SvenskaSpel/locust-plugins/blob/8af21862d8129a5c3b17559677fe92192e312d8f/examples/rest_ex.py#L87) 
* [Use a totally custom load shape/profile](https://docs.locust.io/en/latest/custom-load-shape.html#custom-load-shape)
* [...](https://github.com/locustio/locust/wiki/Extensions)

## Links

* Documentation: [docs.locust.io](https://docs.locust.io)
* Support/Questions: [StackOverflow](https://stackoverflow.com/questions/tagged/locust)
* Github Discussions: [Github Discussions](https://github.com/orgs/locustio/discussions)
* Chat/discussion: [Slack](https://locustio.slack.com) [(signup)](https://communityinviter.com/apps/locustio/locust)

## Authors

* Maintainer: [Lars Holmberg](https://github.com/cyberw)
* UI: [Andrew Baldwin](https://github.com/andrewbaldwin44)
* Original creator: [Jonatan Heyman](https://github.com/heyman)
* Massive thanks to [all of our contributors](https://github.com/locustio/locust/graphs/contributors)

## License

Open source licensed under the MIT license (see _LICENSE_ file for details).
