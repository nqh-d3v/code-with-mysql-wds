Hi mn! Cảm thấy 2 buổi training vừa qua không đủ cho các bạn trong ban lập trình backend hiểu rõ về cách làm việc và lập trình 1 sản phẩm, nên mình làm chuỗi seminar lại cho mn.

Về kiến thức cơ bản, bạn xem lại các tài liệu mà ban lập trình cung cấp nha, ở đây mình sẽ chỉ hướng dẫn flow code hoy.

Đa phần các dự án của CLB sẽ chạy cùng với cơ sở dữ liệu là `mysql`, nên chuỗi bài hướng dẫn bên dưới là dành cho các dự án có sử dụng `mysql`.
# Sub 1

Khi nhìn vào source code của 1 project, source code đa phần có dạng như sau:
```
  ┌ README.md: đây là file mô tả project đó làm cái gì.
  ├ app.js: đây là file sẽ cấu hình để app của bạn chay, app chạy trên port bao nhiu, api nguồn nào sẽ được sử dụng.
  └ src ┬ models: Thư mục này sẽ chạy kèm project khi project bắt đầu chạy. (Giống các ứng dụng Startup trên máy của bạn v đó)
        ├ common: Thư mục này sẽ mô tả các hàm hỗ trợ đi kèm, validate, error-handle,...
        └ api ┬ index.js`: Khai báo các api sẽ được gọi và nó yêu cầu thằng nào sẽ thực hiện thao tác đó
              └ Ngoài ra còn có các module sẽ được gọi từ các api được khai báo trong file `index.js` - [*]
```
Bên trên, mình đã mô tả sơ qua, cấu trúc thư mục của mỗi dự án. Tiếp theo mình sẽ nói rõ hơn về module năm trong thư mục `src/api`.
```
  api ┬ index.js
      ├ account ┬ index.js: File này mô tả các api được gọi sẽ được thực hiện thông qua những hàm controller hoặc hỗ trợ nào.
      │         ├ controllers: File này mô tả các controller sẽ được gọi từ `index.js`
      │         ├ validate: File này mô tả các hàm kiểm tra dữ liệu của các request từ client
      │         ├ services: File mô tả các hàm hỗ trợ và sẽ được gọi từ các controller nếu nó cần thực hiện thao tác liên quan đến module này
      │         └ models ┐ Thư mục này mô tả các mẫu cho module, ví dụ account thì thông tin tài khoản sẽ lưu như nào, gồm thông tin gì,... 
      │                  ├ accounts.js: Mô tả 1 `account` gồm thông tin gì (lưu dữ liệu gì, loại gì, cho phép null hay không,...).
      │                  └ ... Các models khác nếu có.
      └ ... Các module khác
```
-----

Trên đây, mình đã mô tả các file và thư mục trong source code của các dự án có sử dụng `mysql` làm cơ sở dữ liệu. Tiếp theo mình sẽ mô tả, mỗi khi call APIs thì cách xử lý nó sẽ như thế nào.

Ví dụ, mình call 1 api có dạng `https://example.com/api/v1/account/create` với phương thức là `POST`.
```
         CLIENT          │                                              SERVER
                         │
  Client call APIs ──────┼──────> app.js ────────────────> src/api/index.js ──────────────> src/api/accounts/index.js ─────────────┐ 
                         │ Ở đây nó sẽ xem api gốc   - Ở đây nó sẽ xem thằng nào       - Ở đây nó sẽ kiểm tra xem api tiếp         │
                         │ được gọi là gì, thông     được gọi tiếp theo, từ đó         theo được sử dụng (`/create`) có            │
                         │ thường là `api/v1`        gọi module phù hợp thực hiện.     tồn tại không, nếu có thì thực hiện         │
                         │                           - Nó nhận thấy thằng tiếp theo    gọi hàm điều khiển hoặc phải trải qua       │
                         │                           là /account, nó sẽ gọi thằng      hàm kiểm tra (validate) nào.                │
                         │                           nào được khai báo cho `/account`  - Nếu kiểm tra thành công, nó sẽ gọi        │
                         │                                                             hàm điều khiển tương ứng để thực hiện       │
                         │                                                                                                         │
  Response for client <──┼───────── services <───────────────── controllers <────────────────────── validates <────────────────────┘
                         │ - Hàm hỗ trợ tương ứng      - Hàm điều khiển tương ứng được    - Hàm kiểm tra được gọi nhằm kiểm tra  
                         │ được thực hiện và trả       gọi thực hiện, nếu cần thiết, sẽ   dữ liệu gửi đi từ client, nếu oke thì
                         │ giá trị về cho controller.  gọi các hàm hỗ trợ tương ứng để    tiếp tục. Nếu không phù hợp, thì báo lỗi
                         │                             lấy dữ liệu, xử lý nếu có và sau   (400 - Bad Request).
                         │                             đó trả về cho client.
```
Nếu nó kiểm tra và thấy không tồn tại thì trả về lỗi là 404 - Not Found.