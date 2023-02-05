<template>
    <div id="returnPurchaseModal">
        <strong style="font-size: 30px">{{ title }}</strong>
        <table id="returnPurchaseDetail" class="col">
            <colgroup>
                <col width="10%" />
                <col width="10%" />
                <col width="10%" />
                <col width="20%" />
                <col width="10%" />
                <col width="10%" />
                <col width="10%" />
                <col width="20%" />
            </colgroup>
            <thead>
                <tr>
                    <th scope="col">반품번호</th>
                    <th scope="col">납품업체</th>
                    <th scope="col">담당자명</th>
                    <th scope="col">제품명</th>
                    <th scope="col">반품수량</th>
                    <th scope="col">제품단가</th>
                    <th scope="col">합계</th>
                    <th scope="col">창고명</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="item in returnDirectionDetailList" v-bind:key="item.reCode">
                    <td>{{ item.reCode }}</td>
                    <td>{{ item.companyName }}</td>
                    <td>{{ item.name }}</td>
                    <td>{{ item.pdName }}</td>
                    <td>{{ item.reAmt }}</td>
                    <td>{{ item.pdPrice }}</td>
                    <td>{{ item.total }}</td>
                    <td>{{ item.whName }}</td>
                </tr>
            </tbody>
        </table>
        <div class="btn_areaC mt30">
            <a @click="updReturnInfo()" class="btn btn-primary" v-show="!returnIn">
                <span>입금확인</span>
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
        code: Number,
    },
    data: function () {
        return {
            returnDirectionDetailList: [],
            reCode: this.code,
            returnIn: false,
        };
    },
    components: {},
    computed: {},
    // html 로딩, 가상 dom 실행, 이 두 개 연결 시 작동
    mounted: function () {
        /* reCode를 들고 가서 상세내역 가져와서 리스트에 넣어주기 */
        let vm = this;
        let param = new URLSearchParams();

        param.append('reCode', this.reCode);
        param.append('pageNum', 1);
        param.append('listCount', 10);

        this.axios
            .post('/pur/searchReturnPurchase', param)
            .then(function (response) {
                console.log(response.data);

                vm.returnDirectionDetailList = response.data.result.purReturnDirectionList;
            })
            .catch(function (error) {
                alert('에러! API 요청에 오류가 있습니다. ' + error);
            });
    },
    methods: {
        // 발주 - 반품 입금확인
        updReturnInfo: function () {
            console.log('입금하기');

            let vm = this;
            let params = new URLSearchParams();

            params.append('reCode', this.reCode);
            params.append('reOut', '1');
            params.append('pageNum', 1);
            params.append('listCount', 10);

            this.axios
                .post('/pur/updateReturnPurchase', params)
                .then(function (response) {
                    vm.$swal(response.data.result.message);
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
#returnPurchaseDetail {
    width: 66rem;
    margin: 2rem;
    font-size: 1rem;
    height: 6rem;
}
#returnPurchaseModal {
    background-color: #fff;
    padding: 3rem;
    border-radius: 10px;
    border: 2px solid rgb(59, 59, 59);
    text-align: center;
}
</style>
