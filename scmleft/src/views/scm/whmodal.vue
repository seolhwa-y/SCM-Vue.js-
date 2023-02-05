<template>
    <dl>
        <dt>
            <strong>창고등록</strong>
        </dt>
        <dd class="content">
            <!-- s : 여기에 내용입력 -->
            <table class="row">
                <caption>
                    caption
                </caption>

                <tbody>
                    <tr>
                        <th scope="row"><span class="font_red">창고코드</span></th>
                        <td>
                            <input type="text" class="inputTxt p100" v-model="wh_CODE" readonly />
                        </td>
                    </tr>
                    <tr>
                        <th scope="row"><span class="font_red"> 창고명 </span></th>
                        <td>
                            <input
                                type="text"
                                class="inputTxt p100"
                                name="WH_NAME"
                                id="WH_NAME"
                                v-model="wh_NAME" />
                        </td>
                    </tr>
                    <tr>
                        <th scope="row"><span class="font_red"> 우편번호 </span></th>
                        <td>
                            <input
                                type="text"
                                name="wH_Zip"
                                id="wH_Zip"
                                style="height: 25px; width: 360px"
                                v-model="wH_Zip" />
                            <a @click="post()" style="font-size: 13px; float: right; color: blue">
                                우편번호
                            </a>
                        </td>
                    </tr>
                    <tr>
                        <th scope="row"><span class="font_red"> 창고주소 </span></th>
                        <td>
                            <input
                                type="text"
                                class="inputTxt p100"
                                name="wh_Addr"
                                id="wh_Addr"
                                v-model="wh_Addr" />
                        </td>
                    </tr>
                    <tr>
                        <th scope="row"><span class="font_red"> 상세주소 </span></th>
                        <td>
                            <input
                                type="text"
                                class="inputTxt p100"
                                name="wh_ADDR_Detail"
                                id="wh_ADDR_Detail"
                                v-model="wh_ADDR_Detail" />
                        </td>
                    </tr>
                    <tr>
                        <th scope="row"><span class="font_red"> 담당자 </span></th>
                        <select @change="ChangeLoginID($event)">
                            <option v-if="action == 'I'" selected>선택</option>
                            <template v-for="(item, index) in list" :key="item.name">
                                <option v-if="this.name == item.name" v-bind:value="index" selected>
                                    {{ item.name }}
                                </option>
                                <option v-else v-bind:value="index">
                                    {{ item.name }}
                                </option>
                            </template>
                        </select>
                    </tr>
                </tbody>
            </table>
            <!-- class="inputTxt p100" -->
            <!-- e : 여기에 내용입력 -->

            <div class="btn_areaC mt30">
                <a @click="save()" class="btnType blue" id="btnsave" name="btn"
                    ><span>저장</span></a
                >
                <a href="" class="btnType red" id="btndelete" name="btn"><span>삭제</span></a>
                <a @click="closepopup()" class="btnType gray" id="btnClose" name="btn"
                    ><span>닫기</span></a
                >
            </div>
        </dd>
    </dl>
</template>
<script>
import { closeModal } from 'jenesius-vue-modal';

export default {
    props: { whno: Number, action: String, name: String },
    data: function () {
        return {
            pwh_CODE: this.whno,
            wh_CODE: '',
            wh_NAME: '',
            wH_Zip: '',
            wh_Addr: '',
            wh_ADDR_Detail: '',
            list: [],
            loginID: '',
            wh_action: this.action,
            changeName: this.name,
        };
    },
    computed: {},
    mounted: function () {
        let vm = this;

        let params = new URLSearchParams();
        params.append('WH_CODE', this.pwh_CODE);
        //params.append('loginID', this.loginID);

        this.axios
            .post('/scm/detailWh.do', params)
            .then(function (resp) {
                if (vm.pwh_CODE == null || vm.pwh_CODE == '') {
                    vm.wh_CODE = '';
                    vm.wh_NAME = '';
                    vm.wH_Zip = '';
                    vm.wh_Addr = '';
                    vm.wh_ADDR_Detail = '';

                    vm.list = resp.data.WhResponsList;
                } else {
                    console.log(resp.data.WhResponsList);
                    vm.wh_CODE = resp.data.result.wh_CODE;
                    vm.wh_NAME = resp.data.result.wh_NAME;
                    vm.wH_Zip = resp.data.result.wh_ZIP;
                    vm.wh_Addr = resp.data.result.wh_ADDR;
                    vm.wh_ADDR_Detail = resp.data.result.wh_ADDR_DTL;
                    vm.changeName = resp.data.WhResponsList.name;
                    vm.list = resp.data.WhResponsList;
                }
            })
            .catch(function (error) {
                alert('에러 : ' + error);
            });
    },

    methods: {
        post() {
            new window.daum.Postcode({
                oncomplete: (data) => {
                    // 팝업에서 검색결과 항목을 클릭했을때 실행할 코드를 작성하는 부분.

                    // 도로명 주소의 노출 규칙에 따라 주소를 조합한다.
                    // 내려오는 변수가 값이 없는 경우엔 공백('')값을 가지므로, 이를 참고하여 분기 한다.
                    let fullRoadAddr = data.roadAddress; // 도로명 주소 변수
                    let extraRoadAddr = ''; // 도로명 조합형 주소 변수

                    // 법정동명이 있을 경우 추가한다. (법정리는 제외)
                    // 법정동의 경우 마지막 문자가 "동/로/가"로 끝난다.
                    if (data.bname !== '' && /[동|로|가]$/g.test(data.bname)) {
                        extraRoadAddr += data.bname;
                    }
                    // 건물명이 있고, 공동주택일 경우 추가한다.
                    if (data.buildingName !== '' && data.apartment === 'Y') {
                        extraRoadAddr +=
                            extraRoadAddr !== '' ? ', ' + data.buildingName : data.buildingName;
                    }
                    // 도로명, 지번 조합형 주소가 있을 경우, 괄호까지 추가한 최종 문자열을 만든다.
                    if (extraRoadAddr !== '') {
                        extraRoadAddr = ' (' + extraRoadAddr + ')';
                    }
                    // 도로명, 지번 주소의 유무에 따라 해당 조합형 주소를 추가한다.
                    if (fullRoadAddr !== '') {
                        fullRoadAddr += extraRoadAddr;
                    }
                    // 우편번호와 주소 정보를 해당 필드에 넣는다.
                    this.wH_Zip = data.zonecode; //5자리 새우편번호 사용
                    this.wh_Addr = fullRoadAddr;
                    document.getElementById('wh_ADDR_Detail').focus();
                },
            }).open();
        },
        save: function () {
            if (confirm('저장하시겠습니까?')) {
                let params = new URLSearchParams();

                params.append('WH_CODE', this.wh_CODE);
                params.append('WH_NAME', this.wh_NAME);
                params.append('WH_ZIP', this.wH_Zip);
                params.append('WH_ADDR', this.wh_Addr);
                params.append('WH_ADDR_DTL', this.wh_ADDR_Detail);
                params.append('loginID', this.loginID);
                params.append('action', this.wh_action);

                this.axios
                    .post('/scm/WhSave.do', params)
                    .then(function (resp) {
                        console.log(resp);
                        let msg = resp.data.resultMsg;
                        console.log(msg);
                        closeModal();
                    })
                    .catch(function (error) {
                        alert('에러 : ' + error);
                    });
            }
        },
        ChangeLoginID: function (name) {
            console.log(name.target.value);
            //params.append('loginID', this.loginID);
            this.loginID = this.list[name.target.value].loginID;
            console.log(this.loginID);
        },
        closepopup: function () {
            closeModal();
        },
    },
};
</script>

<style>
#searchArea {
    margin-top: 25px;
    margin-bottom: 25px;
}
#searchArea > * {
    height: auto;
}

#groupTitle {
    text-decoration: underline;
    font-weight: bold;
    cursor: pointer;
}
</style>
