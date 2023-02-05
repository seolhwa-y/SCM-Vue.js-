<template>
    <div id="productList">
        <div class="content">
            <p class="Location">
                <a href="/dashboard/home" class="btn_set home">메인으로</a>
                <a href="#" class="btn_nav">거래내역</a>
                <span class="btn_nav bold">일별수주내역</span>
                <a href="../system/comnCodMgr.vue" class="btn_set refresh">새로고침</a>
            </p>
            <p class="conTitle">
                <span>일별수주내역</span>
                <span class="fr">
                    <input type="text" placeholder="검색어를 입력하세요." style="height: 27px" />
                    <a class="btn btn-primary mx-2" v-on:click="searchProductList()">
                        <span>검색</span>
                    </a>
                </span>
            </p>
            <table class="col">
                <colgroup>
                    <col width="7%" />
                    <col width="7%" />
                    <col width="10%" />
                    <col width="10%" />
                    <col width="7%" />
                    <col width="7%" />
                    <col width="7%" />
                    <col width="7%" />
                    <col width="7%" />
                    <col width="7%" />
                    <col width="10%" />
                    <col width="10%" />
                </colgroup>
                <thead>
                    <tr>
                        <th scope="col">주문번호</th>
                        <th scope="col">주문일자</th>
                        <th scope="col">기업명</th>
                        <th scope="col">제품명</th>
                        <th scope="col">재고수량</th>
                        <th scope="col">단가</th>
                        <th scope="col">주문수량</th>
                        <th scope="col">금액합계</th>
                        <th scope="col">반품요청</th>
                        <th scope="col">입금여부</th>
                        <th scope="col">발주지시서</th>
                        <th scope="col">배송지시서</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="item in orderHistoryList" :key="item.jordCode">
                        <td>{{ item.jordCode }}</td>
                        <td>{{ date(item.jordDate) }}</td>
                        <td>{{ item.companyName }}</td>
                        <td>{{ item.pdName }}</td>
                        <td>{{ comma(item.whStock) }}</td>
                        <td>{{ comma(item.pdPrice) }}</td>
                        <td>{{ comma(item.jordAmt) }}</td>
                        <td>{{ comma(item.totalAmt) }}</td>

                        <td v-if="item.reCode != null">Y</td>
                        <td v-else>N</td>

                        <td v-if="item.jordIn == 0">미입금</td>
                        <td v-else>입금</td>

                        <td>
                            <a
                                class="btn btn-primary mx-2"
                                @click="borderDirection(item.jordCode, item.modelCode)"
                                ><span>작성</span></a
                            >
                        </td>

                        <td v-if="item.shCode != 0">작성완료</td>
                        <td v-else>
                            <a
                                class="btn btn-primary mx-2"
                                @click="shipDirection(item.jordCode, item.modelCode, item.jordIn)"
                                ><span>작성</span></a
                            >
                        </td>
                    </tr>
                    <tr v-if="orderHistoryList.length == 0">
                        <td colspan="6">검색된 데이터가 없습니다.</td>
                    </tr>
                </tbody>
            </table>
            <div id="comnGrpCodPagination">
                <paginate
                    class="justify-content-center"
                    v-model="currentPage"
                    :page-count="totalPage"
                    :page-range="10"
                    :margin-pages="0"
                    :click-handler="searchOrderHistoryList"
                    :prev-text="'Prev'"
                    :next-text="'Next'"
                    :container-class="'pagination'"
                    :page-class="'page-item'">
                </paginate>
            </div>
        </div>
    </div>
</template>
<script>
import orderHistoryBorderModal from '@/components/scm/orderHistoryBorderModal.vue';
import orderHistoryShipModal from '@/components/scm/orderHistoryShipModal.vue';
import { openModal } from 'jenesius-vue-modal';
import Paginate from 'vuejs-paginate-next';

export default {
    data: function () {
        return {
            orderHistoryList: [],
            type: '',
            startDate: '',
            endDate: '',
            currentPage: 1, //  현재 페이지
            listCount: 10, // 리스트 뿌릴 사이즈
          //  pageCount: 10, // 페이징 사이즈
            totalPage: 1, // 페이징
            totalCnt: 0, // 리스트의 총 사이즈
        };
    },
    components: {
        paginate: Paginate,
    },
    mounted() {
        /* Vue에서는 el방식이 안먹혀서 mounted에서 search 호출해서 리스트를 뿌려줘야 한다. */
        this.searchOrderHistoryList();
    },
    methods: {
        searchOrderHistoryList: function (pageNum) {
            /* 1. 페이징 시 리스트 뽑아오기
             * 2. 서치바 리스트 뽑아오기
             * 2-1. 서치(주문일자, 반품처리일, 입금여부)
             */
            let vm = this;
            let params = new URLSearchParams();

            if (pageNum != null) this.currentPage = pageNum;

            params.append('pageNum', this.currentPage);
            params.append('listCount', this.listCount);

            this.axios
                .post('/scm/searchOrderHistoryList', params)
                .then(function (response) {
                    console.log(response.data);

                    vm.orderHistoryList = response.data.newOrderSearchList;
                    vm.currentPage = response.data.pageNum;
                    vm.totalCnt = response.data.historyCount;
                    vm.totalPage = vm.page();
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        },
        // 발주 모달 상세 정보 (다중)
        borderDirection: async function (jordCode, modelCode) {
            console.log('borderDirection');

            const modal = await openModal(orderHistoryBorderModal, {
                title: '발주지시서 작성',
                jCode: jordCode,
                mCode: modelCode,
            });

            modal.onclose = (popupparam) => {
                console.log('Close : ' + popupparam);
                this.searchOrderHistoryList(); // 모달이 닫히면서 다시 리스트 뽑아오기
            };
        },
        // 배송 모달 상세 정보
        shipDirection: async function (jordCode, modelCode, jordIn) {
            console.log('shipDirection');

            if (jordIn == 0) return this.$swal('미입금 상태는 배송지시서 작성이 불가능 합니다.');

            const modal = await openModal(orderHistoryShipModal, {
                title: '배송지시서 작성',
                jCode: jordCode,
                mCode: modelCode,
            });

            modal.onclose = (popupparam) => {
                console.log('Close : ' + popupparam);
                this.searchOrderHistoryList(); // 모달이 닫히면서 다시 리스트 뽑아오기
            };
        },
        page: function () {
            var total = this.totalCnt;
            var page = this.listCount;
            var xx = total % page;
            var result = parseInt(total / page);
            if (xx == 0) {
                return result;
            } else {
                result = result + 1;
                return result;
            }
        },
        comma: function (num) {
            return String(num).replace(/\B(?=(\d{3})+(?!\d))/g, ',');
        },
        date: function (date) {
            return date.substring(0, 4) + '-' + date.substring(4, 6) + '-' + date.substring(6, 8);
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
