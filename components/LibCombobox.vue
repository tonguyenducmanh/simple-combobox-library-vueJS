<template lang="">
  <div class="combobox__container">
    <div v-if="hasLabel" class="input__label">
      {{ labelText }}
      <span v-if="showAlertStar" :class="ComboboxEnum.input.LabelAlert">*</span>
    </div>
    <div
      class="combobox"
      :propName="propName"
      :valueName="currentInput"
      v-click-out.passive="onClickOutside"
      @keydown.esc.passive="hideComboboxData"
      @keydown.up.passive="prevEleMove"
      @keydown.left.passive="prevEleMove"
      @keydown.down.passive="nextEleMove"
      @keydown.right.passive="nextEleMove"
      :data-title="dataTitle"
      :value="modelValue"
    >
      <input
        tabindex="0"
        class="combobox__input"
        :class="
          ([classInput],
          isErrorTying === true ? ComboboxEnum.input.InputAlert : '')
        "
        ref="ComboboxInput"
        type="text"
        :placeholder="
          defaultValue !== '' && defaultValue !== undefined
            ? defaultValue
            : placeHolder
        "
        :validate="validate"
        @focus="inputComboboxOnClick"
        @input="
          inputComboboxOnTyping();
          notNullValidate();
        "
        @focusout="notNullValidate"
        v-model="currentInput"
      />
      <button
        tabindex="0"
        class="combobox__button"
        :class="buttonClass"
        @click="btnComboboxOnClick"
      >
        <div class="combobox__drop"></div>
      </button>
      <div
        class="combobox__data"
        ref="hihi"
        :class="[
          isShowData ? ComboboxEnum.comboboxData.SHOW : false,
          isUp ? ComboboxEnum.comboboxData.UP : false,
        ]"
      >
        <!-- itemComboboxOnClick; -->
        <template v-for="(comboboxItem, index) in comboboxList" :key="index">
          <div
            tabindex="0"
            class="combobox__item"
            :value="comboboxItem.value"
            @click="
              $emit('update:modelValue', comboboxItem.value);
              $emit('update:modelName', comboboxItem.name);
              $emit('change-size', comboboxItem.value);
              itemComboboxOnClick();
              notNullValidate();
            "
            @keydown.enter="
              $emit('update:modelValue', comboboxItem.value);
              $emit('update:modelName', comboboxItem.name);
              $emit('change-size', comboboxItem.value);
              itemComboboxOnClick();
              notNullValidate();
            "
            @keydown.tab="
              itemComboboxOnClick(index === comboboxList.length - 1)
            "
            :class="[
              seletedValue === comboboxItem.name
                ? ComboboxEnum.comboboxItem.SELECTED
                : false,
              currentInput !== '' &&
              showAll === false &&
              !comboboxItem.name
                .toLowerCase()
                .includes(currentInput.toLowerCase())
                ? ComboboxEnum.comboboxItem.HIDE
                : false,
            ]"
          >
            {{ comboboxItem.name }}
          </div>
        </template>
      </div>
    </div>
  </div>
</template>
<script>
import ComboboxEnum from "../js/LibEnum.js";
export default {
  name: "LibCombobox",
  props: {
    isUp: Boolean,
    hasLabel: Boolean,
    labelText: String,
    showAlertStar: Boolean,
    classInput: String,
    dataTitle: String,
    validate: String,
    placeHolder: String,
    propName: String,
    defaultValue: String,
    unique: String,
    // giá trị chèn vào khi không có api
    data: String,
    // api để fetch data
    api: String,
    // trường thứ nhất muốn lấy trong json respone
    text: String,
    // trường thứ hai muốn lấy trong json respone
    value: String,
    isNotNull: Boolean,
    isDefaultError: Boolean,
    buttonClass: String,
    // v-model để binding 2 chiều 2 cái dưới vào form bên ngoài combobox nếu có
    modelValue: String,
    modelName: String,
  },
  data() {
    return {
      comboboxList: [],
      ComboboxEnum,
      isShowData: false,
      seletedValue: "",
      currentInput: "",
      uniqueSelected: "",
      isErrorTying: false,
      showAll: true,
    };
  },
  // trường hợp change-size là dùng để emit thay đổi số trang bên EmployeePage
  emits: ["update:modelValue", "update:modelName", "change-size"],
  mounted() {
    /**
     * Tiến hành fetch dữ liệu từ API để chèn vào combobox hoặc
     * phân tách data truyền vào component để tạo ra các comboboxitem
     * Author: Tô Nguyễn Đức Mạnh (11/09/2022)
     */
    try {
      if (this.isDefaultError === true) {
        this.isErrorTying = true;
      }
      let combobox = this;
      // trường hợp có api
      if (this.api !== undefined) {
        fetch(this.api, { method: "GET" })
          .then((res) => res.json())
          .then((res) => {
            // gán giá trị mong muốn vào trong comboboxList
            for (let item of res) {
              // kiểm tra xem nếu có giá trị mặc định thì chèn cho nó vào input và gán value vào combobox
              if (item[combobox.value] === combobox.modelValue) {
                combobox.currentInput = item[combobox.text];
                combobox.isShowData = false;
                // select cái đã chọn
                combobox.seletedValue = item[combobox.text];
                combobox.uniqueSelected = item[combobox.value];
              }
              combobox.comboboxList.push({
                value: item[combobox.value],
                name: item[combobox.text],
              });
            }
          })
          .catch((res) => console.log(res));
      }
      // trường hợp không có api
      else {
        // phân chia các item bằng dấu ;
        let items = this.data.split(";");
        for (let item of items) {
          let arrItem = item.split(":");
          combobox.comboboxList.push({
            value: arrItem[1].trim(),
            name: arrItem[0].trim(),
          });
        }
      }
      // khởi tạo giá trị selected mặc định dựa vào prop truyền vào
      // gán nó vào data đang trống
      if (this.defaultValue !== undefined) {
        this.seletedValue = this.defaultValue;
      }
      // gán value id mặc định
      this.uniqueSelected = this.unique;
    } catch (error) {
      console.log(error);
    }
  },
  methods: {
    /**
     * Validate trường bắt buộc phải nhập (không được để trống)
     * Author: Tô Nguyễn Đức Mạnh (15/09/2022)
     */
    notNullValidate() {
      try {
        // kiểm tra xem có phải trường not null không
        if (
          this.isNotNull === true &&
          (this.currentInput === "" || this.currentInput === undefined)
        ) {
          this.isErrorTying = true;
        } else {
          this.isErrorTying = false;
        }
      } catch (error) {
        console.log(error);
      }
    },
    /**
     * lắng nghe click vào ô input của combobox
     * Author: Tô Nguyễn Đức Mạnh (11/09/2022)
     */
    inputComboboxOnClick() {
      try {
        this.isShowData = true;
        this.showAll = true;
      } catch (error) {
        console.log(error);
      }
    },
    /**
     * lắng nghe nhập liệu vào ô input của combobox
     * Author: Tô Nguyễn Đức Mạnh (11/09/2022)
     */
    inputComboboxOnTyping() {
      try {
        this.isShowData = true;
        this.showAll = false;
      } catch (error) {
        console.log(error);
      }
    },

    /**
     * click vào button thì hiện combobox item.
     * Author: Tô Nguyễn Đức Mạnh (11/09/2022)
     */
    btnComboboxOnClick() {
      try {
        this.isShowData = !this.isShowData;
        this.showAll = true;
      } catch (error) {
        console.log(error);
      }
    },
    /**
     * click vào item thì ẩn combobox data đi và truyền value vào trong input.
     * Author: Tô Nguyễn Đức Mạnh (11/09/2022)
     */
    itemComboboxOnClick(value) {
      try {
        if (value === true || value === undefined || value === "") {
          // ẩn drop menu đi
          this.isShowData = false;
          // select cái đã chọn
          this.seletedValue = event.target.textContent;
          this.currentInput = event.target.textContent;
          this.uniqueSelected = event.target.getAttribute("value");
        }
      } catch (error) {
        console.log(error);
      }
    },
    /**
     * clear toàn bộ value của combobox đi nha
     * Author: Tô Nguyễn Đức Mạnh (14/09/2022)
     */
    clearComboboxSelected() {
      try {
        // ẩn drop menu đi
        this.isShowData = false;
        // select cái đã chọn
        this.seletedValue = "";
        this.currentInput = "";
        this.uniqueSelected = "";
      } catch (error) {
        console.log(error);
      }
    },
    /**
     * click ra ngoài thì ẩn combobox data.
     * Author: Tô Nguyễn Đức Mạnh (11/09/2022)
     */
    onClickOutside() {
      this.isShowData = false;
    },
    /**
     * ấn phím ESC thì ẩn combobox data.
     * Author: Tô Nguyễn Đức Mạnh (11/09/2022)
     */
    hideComboboxData() {
      this.isShowData = false;
    },
    /**
     * Khi ấn lên và sang trái thì di chuyển sang tab index phía trước.
     * Author: Tô Nguyễn Đức Mạnh (11/09/2022)
     */
    prevEleMove(e) {
      if (e.target.previousElementSibling) {
        e.target.previousElementSibling.focus();
      }
    },
    /**
     * Khi ấn xuống và sang phải thì di chuyển sang tab index phía sau.
     * Author: Tô Nguyễn Đức Mạnh (11/09/2022)
     */
    nextEleMove(e) {
      if (e.target.nextElementSibling) {
        e.target.nextElementSibling.focus();
      }
    },
  },
};
</script>
<style scoped>
@import url("../css/LibCombobox.css");
</style>
