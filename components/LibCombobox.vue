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
    // gi?? tr??? ch??n v??o khi kh??ng c?? api
    data: String,
    // api ????? fetch data
    api: String,
    // tr?????ng th??? nh???t mu???n l???y trong json respone
    text: String,
    // tr?????ng th??? hai mu???n l???y trong json respone
    value: String,
    isNotNull: Boolean,
    isDefaultError: Boolean,
    buttonClass: String,
    // v-model ????? binding 2 chi???u 2 c??i d?????i v??o form b??n ngo??i combobox n???u c??
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
  // tr?????ng h???p change-size l?? d??ng ????? emit thay ?????i s??? trang b??n EmployeePage
  emits: ["update:modelValue", "update:modelName", "change-size"],
  mounted() {
    /**
     * Ti???n h??nh fetch d??? li???u t??? API ????? ch??n v??o combobox ho???c
     * ph??n t??ch data truy???n v??o component ????? t???o ra c??c comboboxitem
     * Author: T?? Nguy???n ?????c M???nh (11/09/2022)
     */
    try {
      if (this.isDefaultError === true) {
        this.isErrorTying = true;
      }
      let combobox = this;
      // tr?????ng h???p c?? api
      if (this.api !== undefined) {
        fetch(this.api, { method: "GET" })
          .then((res) => res.json())
          .then((res) => {
            // g??n gi?? tr??? mong mu???n v??o trong comboboxList
            for (let item of res) {
              // ki???m tra xem n???u c?? gi?? tr??? m???c ?????nh th?? ch??n cho n?? v??o input v?? g??n value v??o combobox
              if (item[combobox.value] === combobox.modelValue) {
                combobox.currentInput = item[combobox.text];
                combobox.isShowData = false;
                // select c??i ???? ch???n
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
      // tr?????ng h???p kh??ng c?? api
      else {
        // ph??n chia c??c item b???ng d???u ;
        let items = this.data.split(";");
        for (let item of items) {
          let arrItem = item.split(":");
          combobox.comboboxList.push({
            value: arrItem[1].trim(),
            name: arrItem[0].trim(),
          });
        }
      }
      // kh???i t???o gi?? tr??? selected m???c ?????nh d???a v??o prop truy???n v??o
      // g??n n?? v??o data ??ang tr???ng
      if (this.defaultValue !== undefined) {
        this.seletedValue = this.defaultValue;
      }
      // g??n value id m???c ?????nh
      this.uniqueSelected = this.unique;
    } catch (error) {
      console.log(error);
    }
  },
  methods: {
    /**
     * Validate tr?????ng b???t bu???c ph???i nh???p (kh??ng ???????c ????? tr???ng)
     * Author: T?? Nguy???n ?????c M???nh (15/09/2022)
     */
    notNullValidate() {
      try {
        // ki???m tra xem c?? ph???i tr?????ng not null kh??ng
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
     * l???ng nghe click v??o ?? input c???a combobox
     * Author: T?? Nguy???n ?????c M???nh (11/09/2022)
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
     * l???ng nghe nh???p li???u v??o ?? input c???a combobox
     * Author: T?? Nguy???n ?????c M???nh (11/09/2022)
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
     * click v??o button th?? hi???n combobox item.
     * Author: T?? Nguy???n ?????c M???nh (11/09/2022)
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
     * click v??o item th?? ???n combobox data ??i v?? truy???n value v??o trong input.
     * Author: T?? Nguy???n ?????c M???nh (11/09/2022)
     */
    itemComboboxOnClick(value) {
      try {
        if (value === true || value === undefined || value === "") {
          // ???n drop menu ??i
          this.isShowData = false;
          // select c??i ???? ch???n
          this.seletedValue = event.target.textContent;
          this.currentInput = event.target.textContent;
          this.uniqueSelected = event.target.getAttribute("value");
        }
      } catch (error) {
        console.log(error);
      }
    },
    /**
     * clear to??n b??? value c???a combobox ??i nha
     * Author: T?? Nguy???n ?????c M???nh (14/09/2022)
     */
    clearComboboxSelected() {
      try {
        // ???n drop menu ??i
        this.isShowData = false;
        // select c??i ???? ch???n
        this.seletedValue = "";
        this.currentInput = "";
        this.uniqueSelected = "";
      } catch (error) {
        console.log(error);
      }
    },
    /**
     * click ra ngo??i th?? ???n combobox data.
     * Author: T?? Nguy???n ?????c M???nh (11/09/2022)
     */
    onClickOutside() {
      this.isShowData = false;
    },
    /**
     * ???n ph??m ESC th?? ???n combobox data.
     * Author: T?? Nguy???n ?????c M???nh (11/09/2022)
     */
    hideComboboxData() {
      this.isShowData = false;
    },
    /**
     * Khi ???n l??n v?? sang tr??i th?? di chuy???n sang tab index ph??a tr?????c.
     * Author: T?? Nguy???n ?????c M???nh (11/09/2022)
     */
    prevEleMove(e) {
      if (e.target.previousElementSibling) {
        e.target.previousElementSibling.focus();
      }
    },
    /**
     * Khi ???n xu???ng v?? sang ph???i th?? di chuy???n sang tab index ph??a sau.
     * Author: T?? Nguy???n ?????c M???nh (11/09/2022)
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
