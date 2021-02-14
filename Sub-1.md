Hi mn! Cảm thấy 2 buổi training vừa qua không đủ cho các bạn trong ban lập trình backend hiểu rõ về cách làm việc và lập trình 1 sản phẩm, nên mình làm chuỗi seminar lại cho mn.

Về kiến thức cơ bản, bạn xem lại các tài liệu mà ban lập trình cung cấp nha, ở đây mình sẽ chỉ hướng dẫn flow code hoy.

Đa phần các dự án của CLB sẽ chạy cùng với cơ sở dữ liệu là `mysql`, nên chuỗi bài hướng dẫn bên dưới là dành cho các dự án có sử dụng `mysql`.
# Sub 1
#### Mô tả về source code
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

#### When client call API, what's happen?
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

Một lưu ý nho nhỏ là `controller` của mỗi module chỉ được khai báo để thực hiện thao tác cho module đó, có nghĩa là các `controller` chỉ được khai báo khi module đó cần thực hiện thao tác gì cần gọi đến nó (được xử lý trong index.js). Còn `services` thì cũng giống như vậy, nó được khai báo để thực hiện các thao tác liên quan đến cơ sở dữ liệu cho module hiện tại và có thể được gọi từ các controller hoặc các hàm cần thực hiện các thao tác liên quan đến module đó, ví dụ, khi đăng nhập hoặc đăng ký, `controller` module `auth` sẽ gọi đến service của `account` để lấy thông tin hoặc lưu thông tin tìa khoản vào cơ sở dữ liệu.

#### What is index.js and more...
Dễ thấy trong source có khá nhiều file tên là `index.js`, tại sao phải là `index.js` mà không phải là gì khác. Ngoài ra, trong một số file, còn có  một số dòng khai báo require đến 1 thư mục (ví dụ `const a = require('/c/d/e');`).

Nó có gì liên quan không, mặc định, khi bạn require 1 thư mục, thì nó sẽ tìm đến file `index.js` trong thư mục đó, trong ví dụ trên, có nghĩa là bạn đang require đến file `/c/d/e/index.js`. Và trong `index.js` phải có 1 dòng `export`, lúc này. Bạn bắt đầu rối về require, export và import nó khác nhau như nào đúng không.

Để hiểu rõ về sự khác nhau giữa chúng, bạn tham khảo tại [Import và Export trong JavaScript](https://viblo.asia/p/import-va-export-trong-javascript-maGK7bxM5j2) và [Require vs import](https://viblo.asia/q/phan-biet-cu-phap-require-va-import-aGK7Jooe5j2).

-----

### Bây giờ, khi bạn làm các chức năng cho module thì bạn sẽ cần phải làm gì.
Đối với 1 module, nó sẽ có nhiều chức năng cần phải làm phụ thuộc vào yêu cầu từ phía khách hàng. Về cách code như nào, mình sẽ nói chi tiết ở bài 3.

Thông thường, các bạn trưởng ban sẽ phân tích yêu cầu từ đặc tả mà bạn quản lý dự án giao, thì tùy theo yêu cầu đó, bạn được giao làm phần nào (module nào) thì bạn sẽ phải thực hiện module đó theo yêu cầu từ bạn leader. (Nếu là mình, thông thường mình sẽ mô tả nó trong file README.md).

Bạn đã được học qua Git qua 2 buổi training, thì bạn đã hiểu về cách làm việc với git như thế nào rồi, bạn cũng hiểu git làm gì rồi nên mình sẽ không đề cập nữa.

Ví dụ bạn được giao hoàn thành module `account`, bạn vào tài khoản Gitlab (hoặc GitHub tùy từng dự án) của mình, vào source, nếu chưa được cấp quyền thì nhắn tin cho leader hoặc nhắn trong team dự án. Sau đó bạn cần thực hiện tuần tự các bước sau:
1. Clone dự án về máy của bản thân.
2. Vào đọc phần README để xem cần setup những gì để có thể chạy app, có cần cài đặt biến môi trường gì không,...
3. Tuy vào phần được giao, tạo branch mới ở local và trên Gitlab. (Gõ `git checkout -B account` để tạo branch `account` ở local).
4. Setup app
  - `npm i --save`: Để cài đặt các package cần thiết (hoặc `yarn install` nếu đã cài đặt `yarn`).
  - `npm start`: Để khởi chạy ứng dụng. Nếu bị lỗi, xem lại README xem có đọc thiếu hay gì không. Nếu không thì search Google xem lỗi đó là lỗi gì.

-----

Ở bài tiếp theo mình sẽ nói sơ qua về các thao tác với cơ sở dữ liệu và các phương thức request (GET, POST, PUT, DELETE,...).