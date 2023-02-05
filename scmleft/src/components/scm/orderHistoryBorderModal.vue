<template>
    <div id="orderHistoryBorderModal">
        <strong style="font-size: 30px">{{ title }}</strong>
        <table class="col">
            <thead>
                <tr>
                    <th scope="col">주문번호</th>
                    <th scope="col">제품번호</th>
                    <th scope="col">제품명</th>
                    <th scope="col">제품CODE</th>
                    <th scope="col">납품기업</th>
                </tr>
            </thead>
            <tbody id="borderDrectionTBody">
                <tr>
                    <td>{{ jordCode }}</td>
                    <td>{{ modelCode }}</td>
                    <td>{{ pdName }}</td>
                    <td>{{ pdCode }}</td>
                    <td>{{ companyName }}</td>
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
            ><input type="number" v-model="bordAmt" style="width: 180px; height: 30px" min="0" />
            <a class="btn btn-primary" v-on:click="showBordDetail()"><span>추가</span></a>
        </div>

        <table class="col">
            <thead>
                <tr>
                    <th scope="col">주문번호</th>
                    <th scope="col">제품번호</th>
                    <th scope="col">제품명</th>
                    <th scope="col">제품CODE</th>
                    <th scope="col">납품기업</th>
                    <th scope="col">창고번호</th>
                    <th scope="col">창고명</th>
                    <th scope="col">발주수량</th>
                    <th scope="col">삭제</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(item, index) in bordDetailList" v-bind:key="item.jordCode">
                    <td>{{ item.jordCode }}</td>
                    <td>{{ item.modelCode }}</td>
                    <td>{{ item.pdName }}</td>
                    <td>{{ item.pdCode }}</td>
                    <td>{{ item.companyName }}</td>
                    <td>{{ item.whCode }}</td>
                    <td>{{ item.whName }}</td>
                    <td>{{ item.bordAmt }}</td>
                    <td>
                        <a class="btn btn-danger" v-on:click="delBordDetail(index)"
                            ><span>삭제</span></a
                        >
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
            bordDetailList: [],
            jordCode: this.jCode,
            modelCode: this.mCode,
            pdNams: '',
            companyName: '',
            whCode: '',
            whName: '',
            whStock: '',
            bordAmt: '',
            detailLength: -1,
            loginId: '',
        };
    },
    // html 로딩, 가상 dom 실행, 이 두 개 연결 시 작동
    mounted: function () {
        /* modelCode 들고 가서 상세내역 가져와서 리스트에 넣어주기 */
        let vm = this;
        let param = new URLSearchParams();

        param.append('type', 'b');
        param.append('jordCode', this.jordCode);
        param.append('modelCode', this.modelCode);

        this.axios
            .post('/scm/showDirection', param)
            .then(function (response) {
                console.log(response.data);

                vm.borderList = response.data.bordInfo;
                vm.pdName = response.data.bordInfo.pdName;
                vm.pdCode = response.data.bordInfo.pdCode;
                vm.companyName = response.data.bordInfo.companyName;
                vm.whInfo = response.data.whInfo;
                vm.loginId = response.data.loginId;
            })
            .catch(function (error) {
                alert('에러! API 요청에 오류가 있습니다. ' + error);
            });
    },
    methods: {
        // 지시서 작성완료
        insDirection: function () {
            console.log('insDirection');
            console.log(this.bordDetailList.length);

            let vm = this;
            let direcList = {};
            let param = new URLSearchParams();

            for (let i = 0; i < this.bordDetailList.length; i++) {
                direcList[i] = {};
                direcList[i].jordCode = this.bordDetailList[i].jordCode;
                direcList[i].modelCode = this.bordDetailList[i].modelCode;
                direcList[i].whCode = this.bordDetailList[i].whCode;
                direcList[i].jordAmt = this.bordDetailList[i].bordAmt;
                direcList[i].companyName = this.bordDetailList[i].companyName;
                direcList[i].dirType = '0';
                direcList[i].loginId = this.loginId;
            }

            param.append('list', JSON.stringify(direcList));
            console.log(JSON.stringify(direcList));

            this.axios
                .post('/scm/insertBorderDirection_vue', param)
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
        // 발주지시서 레코드 추가
        showBordDetail: function () {
            console.log('showBordDetail');

            this.detailLength++;
            
            this.bordDetailList[this.detailLength] = [];
            this.bordDetailList[this.detailLength].jordCode = this.jordCode;
            this.bordDetailList[this.detailLength].modelCode = this.modelCode;
            this.bordDetailList[this.detailLength].pdName = this.pdName;
            this.bordDetailList[this.detailLength].pdCode = this.pdCode;
            this.bordDetailList[this.detailLength].companyName = this.companyName;
            this.bordDetailList[this.detailLength].whCode = this.whCode;
            this.bordDetailList[this.detailLength].whName = this.whName;
            this.bordDetailList[this.detailLength].bordAmt = this.bordAmt;

            console.log(this.bordDetailList);
        },
        // 발주지시서 레코드 삭제
        delBordDetail: function (num) {
            console.log('delBordDetail');

            this.bordDetailList.splice(num, 1);
            this.detailLength--;
        },
    },
    created() {
        // 자식요소에서 부모 요소 method 호출
    },
};
</script>

<style>
#orderHistoryBorderModal {
    width: 66rem;
    background-color: #fff;
    padding: 3rem;
    border-radius: 10px;
    border: 2px solid rgb(59, 59, 59);
    text-align: center;
}
</style>
