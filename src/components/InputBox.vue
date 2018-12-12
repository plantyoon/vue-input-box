<template>
  <div class="input-box">
    <label class="wrapper"
      ref="input-wrapper"
      :for="`input-box-${ _uid }`">
      <span class="box-store"
        ref="box-store">
        <span class="box" 
          v-for="(item, index) of store"
          :key="`box-${ index }`">
          <div class="contents"
            @mousedown.prevent="() => {}"
            @click.stop="insert(index)">
            {{ item.value }}
            <button class="close"
              @mousedown.prevent="() => {}"
              @click.stop="remove(index)">🗙</button>
          </div>
        </span>
      </span>
      <input :id="`input-box-${ _uid }`" type="text"
        ref="input"
        v-show="this.store.length < this.limit"
        :style="{ width: inputWidth + 'px' }"
        @input.prevent="consumption"
        @keydown="prevent"
        @keydown.backspace="insert(store.length - 1)"
        @blur="boxing"
        :placeholder="placeholder"
        :value="inputValue" />
      <span class="dummy-input"
        ref="dummy-input">{{ dummyInputValue }}</span>
    </label>
    <div class="alert"
      v-if="alert"
      v-show="isAlert">{{ computedAlertMessage }}</div>
  </div>
</template>

<script>
const preventWords = ['Enter', 'Tab'];
const spliter = ['Enter', 'Tab'];
const lastSpliter = [' ', ','];

export default {
  name: 'InputBox',
  model: {
    event: 'change'
  },
  props: {
    filter: {
      type: String,
      default: '.*',
    },
    limit: {
      type: Number,
      default: Infinity,
    },
    value: {
      type: Array,
      default: () => [],
    },
    placeholder: {
      type: String,
      default: '',
    },
    alert: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      inputValue: '',
      inputWidth: 0,
      cursor: -1,
      isAlert: false,
      store: [],
    };
  },
  mounted() {
    console.log(this);
    this.store.push(...this.value);
    setTimeout(() => this.setInputSize());
  },
  methods: {
    consumption(e) { // 문자 입력 소비 함수
      // console.log('consumption', e);
      const { data, target } = e;
      const { value } = target;
      
      if (value) this.filtering(e);
      else return this.inputValue = value;

      if (!this.totalSpliter.includes(data)) { // 일반 문자 입력

      } else { // 박싱 문자 입력: 박싱 가능하면 박싱 아니면 초기화
        if (this.inputValue) this.boxing(e);
        else this.returnInput();
      }
    },
    prevent(e) { // 금지 문자 입력시 분기 처리 (탭, 엔터 등) - 금지 문자가 아니면 리턴, 금지 문자이면 중단하고 박싱 판단
      console.log('prevent', this.cursor);
      if (preventWords.indexOf(e.key) === -1) return;
      
      if (this.inputValue) {
        this.boxing(e);
      } else {
        if (this.cursor !== -1) this.returnInput();
      }
        // console.log('this.cursor !== -1', this.cursor !== -1, this.cursor)
      if (this.cursor !== -1) { 
        e.preventDefault();
      }
    },
    filtering(e) { // 완성된 문자열 정규식 필터링
      // console.log('filtering');
      const { target, data } = e;
      const { value } = target;
      const tempSelection = target.selectionEnd - (lastSpliter.includes(data) ? 1 : 0);

      if (this.computedFilter.test(value)) {
        this.inputValue = target.value;
        this.isAlert = false;
      }
      else {
        target.value = this.inputValue;
        this.isAlert = true;
      }
      
      target.selectionStart = target.selectionEnd = tempSelection;
    },
    boxing(e) { // 텍스트 박싱
      // console.log('boxing');
      const { target, data, key } = e;
      // console.log('boxing', e);
      if (!this.inputValue || !target.value) return;
      // console.log('boxing go');
      const targetChar = data || key;
      if (spliter.includes(targetChar)) { // 엔터, 탭 박싱문자
        this.addStore({ value: this.inputValue }, this.cursor);
      }
      if (lastSpliter.includes(targetChar)) { // 띄어쓰기, 쉼표 등의 마지막 박싱문자
        if (target.selectionEnd === this.inputValue.length) {
          this.addStore({ value: this.inputValue }, this.cursor);
        } else return;
      }

      if (!targetChar) { // 박싱문자 없이 박싱 (blur 이벤트 등)
        this.addStore({ value: this.inputValue }, this.cursor);
      }
      if (this.cursor !== -1) this.returnInput();
      
      this.$emit('change', this.store);
      
      e.preventDefault();
    },
    addStore(v, i) { // 저장소에 데이터 저장
      // console.log('addStore');
      if (i === -1) this.store.push(v);
      else this.store.splice(i, 0, v);
      this.inputValue = '';
      this.isAlert = false;
    },
    insert(i) { // 데이터 수정시 저장소에서 데이터 로드
      console.log('insert', this.inputValue.length, this.cursor);
      if (this.inputValue.length || this.cursor !== -1) return;
      const targetItem = this.store.splice(i, 1)[0];
      if (!targetItem) return;
      if (this.store.length > i) this.moveInput(i);
      console.log('this.cursor', this.cursor);
      this.$refs['input'].focus();
      this.inputValue = targetItem.value;
    },
    remove(i) { // 저장소에서 데이터 삭제
      // console.log('remove');
      this.store.splice(i, 1);
      setTimeout(() => this.$refs['input'].focus());
      this.$emit('change', this.store);
    },
    moveInput(i) { // 데이터 수정시 해당 위치로 input 이동
      // console.log('moveInput');
      const boxStore = this.$refs['box-store'];
      boxStore.insertBefore(this.$refs['input'], boxStore.children[i]);
      this.cursor = i;
    },
    returnInput() { // 원래 위치로 input 이동
      console.log('returnInput')
      const inputWrapper = this.$refs['input-wrapper'];
      inputWrapper.appendChild(this.$refs['input']);
      this.$refs['input'].focus();
      this.cursor = -1;
    },
    setInputSize() { // 텍스트 길이에 맞게 input 리사이징
      setTimeout(() => {
        const { width } = this.$refs['dummy-input'].getBoundingClientRect();
        this.inputWidth = width + 10;
      });
    },
  },
  computed: {
    totalSpliter() { return [...spliter,...lastSpliter]; },
    dummyInputValue() { return this.inputValue || this.placeholder; },
    computedFilter() { return new RegExp(this.filter); },
    computedAlertMessage() { return this.alert ? this.alert : '' /*|| `The string format is incorrect: ${ this.filter }`;*/ },
  },
  watch: {
    dummyInputValue() { // 새 값이 들어올 때마다 input 리사이징
      // console.log('dummyInputValue');
      this.setInputSize();
    },
  }
}
</script>

<style lang="scss">
$box-color: Salmon;
$inner-padding: 5px;
$box-margin: 2px;
$box-text-color: white;
$input-border: 1px solid #ccc;

.input-box {
  .wrapper {
    display: block;
    box-sizing: border-box;
    padding: $inner-padding;
    border: $input-border;
    cursor: text;
    position: relative;
  }
  input {
    position: relative;
    width: 100%;
    border: 0;
    margin-left: 5px;
    vertical-align: middle;
    outline: none;
  }
  .box-store {
    position: relative;
  }
  .box {
    cursor: pointer;
    display: inline-block;
  }
  .dummy-input {
    opacity: 0;
    position: absolute;
    z-index: -9999;
    white-space: nowrap;
  }
  .alert {
    color: red;
    font-size: 0.8em;
  }
}

.input-box .box {
  padding: $box-margin;
  vertical-align: middle;

  .contents {
    position: relative;
    background: $box-color;
    color: $box-text-color;
    padding-left: 5px;
    line-height: 22px;
    height: 22px;
    border-radius: 5px;
    overflow: hidden;

    &:hover {
      /* background: darken($box-color, 10); */
    }
    .close {
      cursor: pointer;
      background: transparent;
      border: 0;
      color: $box-text-color;
      width: 22px;
      height: 100%;
      float: right;
      border-radius: 5px;
      margin-left: 3px;
      transition: background-color 0.2s;

      &:hover {
        background: darken($box-color, 10);
      }
    }
  }
}

</style>

