<template>
    <dl id="noticeInfoWrap">
        <dd class="content"></dd>
        <!-- s : 여기에 내용입력 -->
        <table id="noticeInfo">
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
                <tr v-if="paction == 'U'">
                    <th scope="row">번호 <span class="font_red">*</span></th>
                    <td>
                        <input type="text" class="form-control" readonly v-model="notCode" />
                    </td>
                    <th scope="row">작성자</th>
                    <td>
                        <input type="text" class="form-control" readonly v-model="name" />
                        <input type="hidden" v-model="loginId" />
                    </td>
                </tr>
                <tr>
                    <th scope="row">제목<span class="font_red">*</span></th>
                    <td colspan="3">
                        <input
                            type="text"
                            class="form-control"
                            v-model="notTitle"
                            v-bind:readonly="!readType" />
                    </td>
                </tr>
                <tr>
                    <th scope="row">내용<span class="font_red">*</span></th>
                    <td colspan="3">
                        <textarea v-model="notCon" v-bind:readonly="!readType"></textarea>
                    </td>
                </tr>
            </tbody>
        </table>

        <!-- e : 여기에 내용입력 -->

        <div class="btn_areaC mt30">
            <a @click="saveNotice()" class="btn btn-primary" v-show="savShow">
                <span>저장</span>
            </a>
            <a @click="regNotice()" class="btn btn-primary" v-show="regShow">
                <span>수정</span>
            </a>
            <a @click="delNotice()" class="btn btn-danger mx-2" v-show="delShow">
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
        code: Number,
        id: String,
        action: String,
    },
    data: function () {
        return {
            notCode: this.code,
            paction: this.action,
            loginId: this.id,
            writeId: '',
            notTitle: '',
            notCon: '',
            notDate: '',
            name: '',
            savShow: true,
            regShow: true,
            delshow: true,
            readType: false,
        };
    },
    computed: {},
    // html 로딩, 가상 dom 실행, 이 두 개 연결 시 작동
    mounted: function () {
        let vm = this;
        // 신규 등록 시
        if (this.notCode == null || this.notCode == '') {
            vm.notCode = '';
            vm.notTitle = '';
            vm.notCon = '';
            vm.notDate = '';
            vm.loginId = '';
            vm.name = '';
            vm.regShow = false;
            vm.delShow = false;
            vm.readType = true;
        } else {
            //  수정 시 (grp_cod 에 해당하는 상세코드 정보 가져오기)
            let params = new URLSearchParams();
            params.append('notCode', this.notCode);
            params.append('view', 'Y');

            this.axios
                .post('/scm/checkNoticeList', params)
                .then(function (response) {
                    console.log(response);
                    console.log(response.data);

                    vm.notCode = response.data.result.noticeList[0].notCode;
                    vm.notTitle = response.data.result.noticeList[0].notTitle;
                    vm.notCon = response.data.result.noticeList[0].notCon;
                    vm.notDate = response.data.result.noticeList[0].notDate;
                    vm.name = response.data.result.noticeList[0].name;
                    vm.writeId = response.data.result.noticeList[0].loginId;

                    // 작성자와 로그인한 유저가 같은지 판별에 따라 버튼 SHOW 유무 판별
                    if (vm.loginId != vm.writeId) {
                        vm.savShow = false;
                        vm.regShow = false;
                        vm.delShow = false;
                    } else if (vm.loginId == vm.writeId) {
                        vm.savShow = false;
                        vm.delShow = true;
                    }
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        }
    },
    methods: {
        // 수정 버튼 클릭시 입력창 활성화
        regNotice: function () {
            this.readType = true;
            this.savShow = true;
            this.regShow = false;
            this.delShow = false;
        },
        saveNotice: function () {
            if (confirm('저장하시겠습니까?')) {
                let vm = this;
                let params = new URLSearchParams();
                let jobCode = '/scm/insertNotice';

                if (this.action != 'I') params.append('notCode', this.notCode);
                params.append('notTitle', this.notTitle);
                params.append('notCon', this.notCon);

                if (this.action != 'I') jobCode = '/scm/updateNotice';

                this.axios
                    .post(jobCode, params)
                    .then(function (response) {
                        console.log(response);

                        let status = response.status;

                        if (status == 200) {
                            closeModal(vm);
                            vm.$swal(response.data.result.message);
                        } else {
                            vm.$swal(response.data.result.message);
                        }
                    })
                    .catch(function (error) {
                        alert('에러! API 요청에 오류가 있습니다. ' + error);
                    });
            }
        },
        delNotice: function () {
            let vm = this;
            let params = new URLSearchParams();
            params.append('notCode', this.notCode);

            this.axios
                .post('/scm/deleteNotice', params)
                .then(function (response) {
                    console.log(response);

                    closeModal(vm);
                    vm.$swal(response.data.result.message);
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
#noticeInfo {
    border-collapse: separate;
    border-spacing: 20px;
}
#noticeInfoWrap {
    background-color: #fff;
    padding: 3rem;
    border-radius: 10px;
    border: 2px solid rgb(59, 59, 59);
}
</style>
