<template>
    <div id="returnPuchase">
        <div class="content">
            <p class="Location">
                <a href="/dashboard/home" class="btn_set home">메인으로</a>
                <a href="#" class="btn_nav">납품업체</a>
                <span class="btn_nav bold">반품지시서 목록</span>
                <a href="../system/comnCodMgr.vue" class="btn_set refresh">새로고침</a>
            </p>

            <!-- 검색 영역 -->
            <div id="divSearchBar">
                <p class="conTitle">
                    <span>반품지시서 목록</span>
                    <span class="fr">
                        <select id="searchKey" v-model="search">
                            <option value="all" selected="selected">전체</option>
                            <option value="companyName">업체명</option>
                            <option value="pdName">제품명</option>
                        </select>
                        <input type="date" v-model="startDate" />
                        <input type="date" v-model="endDate" />
                        <input
                            type="text"
                            v-model="keyword"
                            placeholder="검색어를 입력하세요."
                            style="height: 27px" />
                        <a
                            class="btn btn-primary"
                            style="color: white"
                            v-on:click="searchBordReturnList()">
                            <span>검색</span>
                        </a>
                    </span>
                </p>
            </div>

            <!-- 반지시서 리스트 영역 -->
            <div id="divPurReturnList">
                <table class="col">
                    <colgroup>
                        <col width="10%" />
                        <col width="20%" />
                        <col width="40%" />
                        <col width="15%" />
                        <col width="15%" />
                    </colgroup>
                    <thead>
                        <tr>
                            <th scope="col">반품번호</th>
                            <th scope="col">납품업체</th>
                            <th scope="col">제품명</th>
                            <th scope="col">날짜</th>
                            <th scope="col">여부</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="item in purReturnDirectionList" :key="item.reCode">
                            <td>{{ item.reCode }}</td>
                            <td>{{ item.companyName }}</td>
                            <td>{{ item.pdName }}</td>
                            <td>{{ item.dirDate }}</td>
                            <td v-if="item.reType == '2'" style="color: red; font-weight: bold">
                                {{ item.typeName }}
                            </td>
                            <td
                                v-else-if="item.reType == '1' && item.reOut == '1'"
                                style="color: red; font-weight: bold">
                                입금완료
                            </td>
                            <td v-else-if="item.reType == 1">
                                <a
                                    class="btn btn-primary"
                                    v-on:click="showReturnDirectionDetail(item.reCode)"
                                    ><span>입금</span></a
                                >
                            </td>
                        </tr>
                        <tr v-if="purReturnDirectionList.length == 0">
                            <td colspan="5">검색된 데이터가 없습니다.</td>
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
                        :click-handler="searchBordReturnList"
                        :prev-text="'Prev'"
                        :next-text="'Next'"
                        :container-class="'pagination'"
                        :page-class="'page-item'">
                    </paginate>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import returnPuchaseModal from '@/components/pur/returnPurchaseModal.vue';
import { openModal } from 'jenesius-vue-modal';
import Paginate from 'vuejs-paginate-next';

export default {
    data: function () {
        return {
            purReturnDirectionList: [],
            search: 'all',
            startDate: '',
            endDate: '',
            keyword: '',
            currentPage: 1, //  현재 페이지
            listCount: 10, // 리스트 뿌릴 사이즈
            pageCount: 10, // 페이징 사이즈
            totalPage: 1, // 페이징
            totalCnt: 0, // 리스트의 총 사이즈
        };
    },
    components: {
        paginate: Paginate,
    },
    mounted() {
        /* Vue에서는 el방식이 안먹혀서 mounted에서 search 호출해서 리스트를 뿌려줘야 한다. */
        this.searchBordReturnList();
    },
    methods: {
        searchBordReturnList: function (pageNum) {
            /* 1. 페이징 시 리스트 뽑아오기
             * 2. 서치바 리스트 뽑아오기
             * 2-1. 서치(주문일자, 반품처리일, 입금여부)
             * 3. 날짜 유효서 체크
             */
            if (this.endDate.replaceAll('-', '') - this.startDate.replaceAll('-', '') < 0)
                return this.$swal('날짜를 다시 선택해주세요.');

            let vm = this;
            let params = new URLSearchParams();

            if (pageNum != null) this.currentPage = pageNum;
            params.append('pageNum', this.currentPage);
            params.append('listCount', this.listCount);
            params.append('type', this.search);
            params.append('keyword', this.keyword);
            params.append('startDate', this.startDate.replaceAll('-', ''));
            params.append('endDate', this.endDate.replaceAll('-', ''));

            this.axios
                .post('/pur/searchReturnPurchase', params)
                .then(function (response) {
                    console.log(response.data);
                    console.log(response.data.result.purReturnDirectionCount);

                    vm.currentPage = parseInt(response.data.result.pageNum);
                    vm.purReturnDirectionList = response.data.result.purReturnDirectionList;
                    vm.totalCnt = response.data.result.purReturnDirectionCount;
                    vm.totalPage = vm.page();
                    vm.search = 'all';
                    vm.keyword = '';
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        },
        // 발주 - 반품지시서 상세내역 모달 열기
        showReturnDirectionDetail: async function (reCode) {
            /* reCode를 들고 modal에 전근하여 back에서 데이터 들고 내용 채우기 */
            console.log('발주 - 반품지시서');

            const modal = await openModal(returnPuchaseModal, {
                title: '반품지시서 상세보기',
                code: reCode,
                pageNum: this.currentPage,
                listCount: this.listCount,
            });

            modal.onclose = (popupparam) => {
                console.log('Close : ' + popupparam);
                this.searchBordReturnList(); // 모달이 닫히면서 다시 리스트 뽑아오기
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
    },
};
</script>

<style>
#groupTitle {
    text-decoration: underline;
    font-weight: bold;
    cursor: pointer;
}
</style>
