<template>
    <div id="orderHistoryShipModal">
        <strong style="font-size: 30px">{{ title }}</strong>
        <table class="col">
            <thead>
                <tr>
                    <th scope="col">주문번호</th>
                    <th scope="col">주문일자</th>
                    <th scope="col">기업명</th>
                    <th scope="col">제품번호</th>
                    <th scope="col">제품명</th>
                    <th scope="col">제품CODE</th>
                    <th scope="col">주문수량</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>{{ jordCode }}</td>
                    <td>{{ jordDate }}</td>
                    <td>{{ companyName }}</td>
                    <td>{{ modelCode }}</td>
                    <td>{{ pdName }}</td>
                    <td>{{ pdCode }}</td>
                    <td>{{ jordAmt }}</td>
                </tr>
            </tbody>
        </table>

        <div
            id="whInfo"
            style="
                margin-bottom: 1rem;
                display: flex;
                justify-content: space-around;
                align-items: baseline;
            ">
            <span>창고선택</span>
            <select v-on:change="getWhStock($event)">
                <option selected disabled>창고선택</option>
                <option v-for="(item, index) in whInfo" :key="index" v-bind:value="index">
                    {{ item.whName }}
                </option>
            </select>

            <span>재고수량</span
            ><input
                v-model="whStock"
                type="text"
                readonly="readonly"
                style="width: 180px; height: 30px" />
            <span>수량</span
            ><input type="number" v-model="shipAmt" style="width: 180px; height: 30px" min="0" />
            <a class="btn btn-primary" v-on:click="showShipDetail()"><span>추가</span></a>
        </div>

        <table class="col">
            <thead>
                <tr>
                    <th scope="col">주문번호</th>
                    <th scope="col">기업명</th>
                    <th scope="col">제품번호</th>
                    <th scope="col">제품명</th>
                    <th scope="col">제품CODE</th>
                    <th scope="col">창고번호</th>
                    <th scope="col">창고명</th>
                    <th scope="col">배송수량</th>
                    <th scope="col">배송담당자</th>
                    <th scope="col">삭제</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="item in shipDetailList" v-bind:key="item.jordCode">
                    <td>{{ item.jordCode }}</td>
                    <td>{{ item.companyName }}</td>
                    <td>{{ item.modelCode }}</td>
                    <td>{{ item.pdName }}</td>
                    <td>{{ item.pdCode }}</td>
                    <td>{{ item.whCode }}</td>
                    <td>{{ item.whName }}</td>
                    <td>{{ item.shipAmt }}</td>
                    <td>
                        <select v-on:change="getShipName($event)">
                            <option selected disabled>배송담당자 선택</option>
                            <option
                                v-for="(item, index) in deliInfo"
                                :key="index"
                                v-bind:value="index">
                                {{ item.deliName }}
                            </option>
                        </select>
                    </td>
                    <td>
                        <a class="btn btn-danger" v-on:click="delShipDetail()"><span>삭제</span></a>
                    </td>
                </tr>
            </tbody>
        </table>

        <div class="btn_areaC mt30">
            <a @click="insDirection()" class="btn btn-primary">
                <span>작성완료</span>
            </a>
        </div>
    </div>
</template>
<script>
import { closeModal } from 'jenesius-vue-modal';

export default {
    // vue에서는 받아온 변수를 methods에서 직접 핸들링이 불가능하기 때문에
    // 임시 변수를 만들어서 받아온 변수를 넣어 줘야 함
    props: {
        title: String,
        jCode: Number,
        mCode: Number,
    },
    data: function () {
        return {
            whInfo: [],
            shipDetailList: [],
            shipList: [],
            deliInfo: [],
            jordCode: this.jCode,
            modelCode: this.mCode,
            pdNams: '',
            companyName: '',
            whCode: '',
            whName: '',
            whStock: '',
            deliCode: '',
            deliName: '',
            shipAmt: '',
            detailLength: -1,
            loginId: '',
        };
    },
    components: {},
    computed: {},
    // html 로딩, 가상 dom 실행, 이 두 개 연결 시 작동
    mounted: function () {
        /* modelCode 들고 가서 상세내역 가져와서 리스트에 넣어주기 */
        let vm = this;
        let param = new URLSearchParams();

        param.append('type', 's');
        param.append('jordCode', this.jordCode);
        param.append('modelCode', this.modelCode);

        this.axios
            .post('/scm/showDirection', param)
            .then(function (response) {
                console.log(response.data);

                vm.whInfo = response.data.whInfo;
                vm.shipList = response.data.shipInfo;
                vm.pdName = response.data.shipInfo.pdName;
                vm.pdCode = response.data.shipInfo.pdCode;
                vm.jordDate = response.data.shipInfo.jordDate;
                vm.companyName = response.data.shipInfo.companyName;
                vm.pdName = response.data.shipInfo.pdName;
                vm.jordAmt = response.data.shipInfo.jordAmt;
                vm.deliInfo = response.data.deliInfo;

                console.log(vm.shipDetailList);
            })
            .catch(function (error) {
                alert('에러! API 요청에 오류가 있습니다. ' + error);
            });
    },
    methods: {
        // 지시서 작성완료
        insDirection: function () {
            let vm = this;
            let param = new URLSearchParams();

            if (this.checkValue() != true) return;
            if (this.deliCode == '') return this.$swal('', '배송담당자를 선택하세요.', 'error');

            param.append('dirType', '2');
            param.append('jordCode', this.jordCode);
            param.append('modelCode', this.modelCode);
            param.append('companyName', this.companyName);
            param.append('whCode', this.whCode);
            param.append('jordAmt', this.shipAmt);
            param.append('deliID', this.deliCode);

            this.axios
                .post('/scm/insertShipDirection', param)
                .then(function (response) {
                    console.log(response);

                    closeModal(vm);
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        },
        // 창고별 재고 표시
        getWhStock: function (num) {
            console.log('getWhStock');
            console.log(num.target.value);

            this.whCode = this.whInfo[num.target.value].whCode;
            this.whName = this.whInfo[num.target.value].whName;
            this.whStock = this.whInfo[num.target.value].whStock;
        },
        getShipName: function (num) {
            console.log('getShipName');
            console.log(num.target.value);

            this.deliCode = this.deliInfo[num.target.value].loginId;
            this.deliName = this.deliInfo[num.target.value].deliName;
        },
        // 배송지시서 레코드 추가
        showShipDetail: function () {
            console.log('showBordDetail');

            if (this.checkValue() != true) return;

            if (this.shipDetailList.length == 1)
                return this.$swal('배송지시서는 한건만 추가 가능합니다.');

            this.shipDetailList[0] = [];
            this.shipDetailList[0].jordCode = this.jordCode;
            this.shipDetailList[0].modelCode = this.modelCode;
            this.shipDetailList[0].pdName = this.pdName;
            this.shipDetailList[0].pdCode = this.pdCode;
            this.shipDetailList[0].companyName = this.companyName;
            this.shipDetailList[0].whCode = this.whCode;
            this.shipDetailList[0].whName = this.whName;
            this.shipDetailList[0].shipAmt = this.shipAmt;
        },
        // 배송지시서 레코드 삭제
        delShipDetail: function () {
            console.log('delBordDetail');
            console.log(this.shipDetailList);

            this.shipDetailList.splice(0, 1);
        },
        // 값 체크
        checkValue: function () {
            let check = false;

            if (this.whCode == '') return this.$swal('', '창고를 선택하세요.', 'error');
            if (this.shipAmt == '' || this.shipAmt == 0)
                return this.$swal('', '수량을 입력하세요.', 'error');
            if (this.whStock - this.shipAmt < 0)
                return this.$swal(
                    '',
                    '해당 창고의 재고수량보다 수량이 많습니다. 다른 창고를 선택하거나 발주지시서를 작성하세요.',
                    'error'
                );
            if (this.jordAmt != this.shipAmt)
                return this.$swal('', '주문수량과 같지 않습니다. 다시 작성하세요.', 'error');

            check = true;

            return check;
        },
    },

    created() {
        // 자식요소에서 부모 요소 method 호출
    },
};
</script>

<style>
#orderHistoryShipModal {
    width: 66rem;
    background-color: #fff;
    padding: 3rem;
    border-radius: 10px;
    border: 2px solid rgb(59, 59, 59);
    text-align: center;
}
</style>
