## Thư viện tạo combobox từ component VueJS đơn giản

Code sẽ được viết trong Component có dạng như sau

```html
có api
<LibCombobox
  :hasLabel="true"
  labelText="Đơn vị"
  :showAlertStar="true"
  dataTitle="Đơn vị không được để trống."
  :api="MISAEnum.API.GETDEPARTMENTLIST"
  text="departmentName"
  value="departmentID"
  class="form__ele"
  placeHolder="Nhập đơn vị"
  classInput="input__musthave"
  buttonClass="combobox__button--white"
  v-model:modelValue="formObject['departmentID']"
  v-model:modelName="formObject['departmentName']"
  unique=""
  :isNotNull="true"
/>
không có api
<LibCombobox
  id="page_ranges"
  classInput="input__field"
  propName="PageNavigation"
  :data="10 bản ghi trên trang:10;20 bản ghi trên trang:20;50 bản ghi trên trang: 50;100 bản ghi trên trang:100"
  placeHolder="Hãy chọn số trang"
  :defaultValue="10 bản ghi trên trang"
  :unique="10"
  :isUp="true"
  @change-size="changeSize"
/>
```

Trong đó : (ngoại trừ data, text, value, api tất cả áp dụng cho cả 2 trường hợp)

- v-model:modelValue : giá trị ID muốn lấy khi click vào combobox
- v-model:modelName : giá trị value muốn lấy khi click vào combobox
  dùng 2 cái trên để binding 2 chiều cùng form
- id: tên id của combobox muốn đặt
- api: địa chỉ api của trang web fetch data về
- text : giá trị của trường muốn lấy trong json trả về để gán vào textContent item
- value: giá trị của trường muốn lấy để gán vào value của item
- unique: giá trị mặc định được set bên phải value
  ( ví dụ ta có : 50 sản phẩm : 50 => value là 50 sản phẩm, unique là 50)
- placeHolder: giá trị gợi ý khi chưa nhập vào input
- data: giá trị thay cho api, viết dạng Text1: value1;Text2: value2;Text3:valu3
- defaultValue: chỉ dùng khi có data, giá trị được chọn mặc định
- propName: tên riêng đặt cho combobox này\
  Options :
- classInput: tên class của input muốn đặt
- dataTitle: dùng như tooltip, hover vào sẽ hiện lên
- validate: giá trị của trường validate, dùng để gọi các resource dựa vào tên validate này
- isUp : trả về true thì combobox data sổ lên, false sổ xuống.
- isNotNull: trả về true thì trường này bắt buộc nhập,nếu không nhập sẽ báo đỏ
- isDefaultError: trả về true thì trường này sẽ báo đỏ ngay lúc mounted
- buttonClass="combobox__button--white": class tùy chỉnh cho nút btn, giả dụ muốn đổi màu chẳng hạn

Ngoài ra có thể sử dụng phím ESC để ẩn combobox
Phím Enter để nhập giá trị vào input và value vào combobox
Các phím lên xuống trái phải để điều hướng khi ấn tab index
