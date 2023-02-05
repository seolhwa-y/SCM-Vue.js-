<template>
    <dl id="grpInfoWrap">
        <dd class="content"></dd>
        <!-- s : 여기에 내용입력 -->
        <table id="grpInfo">
            <caption>
                caption
            </caption>
            <colgroup>
                <col width="120px" />
                <col width="*" />
                <col width="120px" />
                <col width="*" />
            </colgroup>

            <tbody>
                <tr>
                    <td colspan="4" class="text-center">
                        <div class="my-4">
                            <strong style="font-size: 30px">{{ title }}</strong>
                        </div>
                    </td>
                </tr>

                <tr>
                    <th scope="row">그룹 코드 <span class="font_red">*</span></th>
                    <td>
                        <input
                            type="text"
                            class="form-control"
                            name="grp_cod"
                            id="grp_cod"
                            v-model="dtl_grp_cod"
                            :readonly="readText" />
                    </td>
                    <th scope="row">그룹 코드 명 <span class="font_red">*</span></th>
                    <td>
                        <input
                            type="text"
                            class="form-control"
                            name="grp_cod_nm"
                            id="grp_cod_nm"
                            v-model="dtl_grp_cod_nm"
                            :readonly="readText" />
                    </td>
                </tr>

                <tr>
                    <th scope="row">상세 코드</th>
                    <td>
                        <input
                            type="text"
                            class="form-control"
                            name="grp_cod_eplti"
                            id="grp_cod_eplti"
                            v-model="dtl_cod"
                            :readonly="readText" />
                    </td>
                    <th scope="row">상세 코드 명</th>
                    <td>
                        <input
                            type="text"
                            class="form-control"
                            name="grp_cod_eplti"
                            id="grp_cod_eplti"
                            v-model="dtl_cod_nm" />
                    </td>
                </tr>

                <tr>
                    <th scope="row">사용 유무 <span class="font_red">*</span></th>
                    <td colspan="3">
                        <input
                            type="radio"
                            name="grp_use_poa"
                            id="grp_use_poa_1"
                            value="Y"
                            v-model="dtl_use_poa" />
                        <label for="radio1-1">사용</label>
                        &nbsp;&nbsp;&nbsp;&nbsp;
                        <input
                            type="radio"
                            name="grp_use_poa"
                            id="grp_use_poa_2"
                            value="N"
                            v-model="dtl_use_poa" />
                        <label for="radio1-2">미사용</label>
                    </td>
                </tr>
            </tbody>
        </table>

        <!-- e : 여기에 내용입력 -->

        <div class="btn_areaC mt30">
            <a @click="save()" class="btn btn-primary" id="btnSaveGrpCod" name="btn">
                <span>저장</span>
            </a>
            <a @click="deletefrpcd()" class="btn btn-danger mx-2" v-show="delshow">
                <span>삭제</span>
            </a>
        </div>
    </dl>
</template>

<script>
import { closeModal } from 'jenesius-vue-modal';
export default {
    // vue에서는 받아온 변수를 methods에서 직접 핸들링이 불가능하기 때문에
    // 임시 변수를 만들어서 받아온 변수를 넣어 줘야 함
    props: {
        title: String,
        grpcd: String,
        grpnm: String,
        detailCode: String,
        detailName: String,
        detailUse: String,
        action: String,
    },
    data: function () {
        return {
            dtl_grp_cod: this.grpcd,
            dtl_grp_cod_nm: this.grpnm,
            dtl_cod: this.detailCode,
            dtl_cod_nm: this.detailName,
            dtl_use_poa: '',
            delshow: false,
            eventYn: '',
            readText: false,
        };
    },
    computed: {
        agrpcd: {
            get: function () {
                return this.data.pgrpcd;
            },
            set: function (v) {
                this.data.pgrpcd = v;
            },
        },
    },
    // html 로딩, 가상 dom 실행, 이 두 개 연결 시 작동
    mounted: function () {
        //let vm = this;
        if (this.action == 'I') this.readText = false;
        else {
            this.readText = true;
            this.dtl_use_poa = this.detailUse;
            this.delshow = true;
        }
    },
    methods: {
        save: function () {
            if (confirm('저장하시겠습니까?')) {
                let vm = this;
                let params = new URLSearchParams();

                params.append('dtl_grp_cod', this.dtl_grp_cod);
                params.append('grp_cod_nm', this.dtl_grp_cod_nm);
                params.append('dtl_cod', this.dtl_cod);
                params.append('dtl_cod_nm', this.dtl_cod_nm);
                params.append('dtl_use_poa', this.dtl_use_poa);
                params.append('action', this.action);

                console.log(params);

                this.axios
                    .post('/system/saveComnDtlCod.do', params)
                    .then(function (response) {
                        console.log(response);
                        let status = response.status;
                        let msg = response.data.resultMsg;
                        if (status == 200) {
                            vm.$swal(msg);
                            closeModal(vm);
                        } else {
                            vm.$swal(msg);
                        }
                    })
                    .catch(function (error) {
                        alert('에러! API 요청에 오류가 있습니다. ' + error);
                    });
            }
        },
        deletefrpcd: function () {
            let vm = this;
            let params = new URLSearchParams();

            params.append('dtl_grp_cod', this.dtl_grp_cod);
            params.append('dtl_cod', this.dtl_cod);

            this.axios
                .post('/system/deleteComnDtlCod.do', params)
                .then(function (response) {
                    console.log(response);
                    vm.$swal(response.data.resultMsg);
                    closeModal(vm);
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        },
    },
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
