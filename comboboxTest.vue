<template lang="">
  <div>đây là trang demo sử dụng combobox</div>
  <div>trường hợp có api</div>
  <LibCombobox
    api="https://cukcuk.manhnv.net/api/v1/Departments"
    text="DepartmentName"
    value="DepartmentId"
    placeHolder="Hãy chọn phòng ban"
  />
  <br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br />
  <div>trường hợp không có api</div>
  <LibCombobox
    data="10 bản ghi trên trang:10;20 bản ghi trên trang:20;50 bản ghi trên trang: 50;100 bản ghi trên trang:100"
    placeHolder="Hãy chọn số trang"
    defaultValue="10 bản ghi trên trang"
    unique="10"
  />
</template>
<script>
import LibCombobox from "./components/LibCombobox.vue";
export default {
  name: "comboboxTest",
  components: {
    LibCombobox,
  },
};
</script>
<style lang=""></style>
