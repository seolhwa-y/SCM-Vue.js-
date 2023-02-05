<template>
  <select
    :id="sid"
    :name="sid"
    v-model="defaultvalue"
    :modelvalue="defaultvalue"
  >
    <template v-if="type == 'all'">
      <option value="">전체</option>
    </template>
    <template v-else>
      <option value="">선택</option>
    </template>
    <option v-for="itemone in items" :value="itemone.cd" :key="itemone.cd">
      {{ itemone.name }}
    </option>
  </select>
</template>

<script>
export default {
  // vue에서는 받아온 변수를 methods에서 직접 핸들링이 불가능하기 때문에
  // 임시 변수를 만들어서 받아온 변수를 넣어 줘야 함
  props: {
    comtype: String,
    selectid: String,
    type: String,
    selvalue: String,
  },
  data: function () {
    return {
      items: [],
      defaultvalue: '',
      sid: '',
    };
  },
  // html 로딩, 가상 dom 실행, 이 두 개 연결 시 작동
  mounted: function () {
    let vm = this;
    this.sid = this.selectid;

    //  수정 시 (grp_cod 에 해당하는 상세코드 정보 가져오기)
    let params = new URLSearchParams();
    params.append('comtype', this.comtype);

    this.axios
      .post('/system/selectComCombo.do', params)
      .then(function (response) {
        console.log(response);

        vm.items = response.data.list;
        vm.defaultvalue = '';
      })
      .catch(function (error) {
        alert('에러! API 요청에 오류가 있습니다. ' + error);
      });
    this.$emit('send-com-message', this.defaultvalue);
  },
  methods: {},
  created() {
    // 자식요소에서 부모 요소 method 호출
  },
};
</script>

<style>
#grpInfo {
  border-collapse: separate;
  border-spacing: 20px;
}
#grpInfoWrap {
  background-color: #fff;
  padding: 3rem;
  border-radius: 10px;
  border: 2px solid rgb(59, 59, 59);
}
</style>
