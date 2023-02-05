<template>
    <div id="productList">
        <div class="content">
            <p class="Location">
                <a href="/dashboard/home" class="btn_set home">메인으로</a>
                <a href="#" class="btn_nav">주문</a>
                <span class="btn_nav bold">제품 목록</span>
                <a href="../system/comnCodMgr.vue" class="btn_set refresh">새로고침</a>
            </p>
            <p class="conTitle">
                <span>제품 목록</span>
                <span class="fr">
                    <input
                        type="text"
                        v-model="keyword"
                        placeholder="검색어를 입력하세요."
                        style="height: 27px" />
                    <a class="btn btn-primary mx-2" v-on:click="searchProductList()">
                        <span>검색</span>
                    </a>
                </span>
            </p>
            <table class="col">
                <colgroup>
                    <col width="10%" />
                    <col width="10%" />
                    <col width="20%" />
                    <col width="30%" />
                    <col width="15%" />
                </colgroup>
                <thead>
                    <tr>
                        <th scope="col">모델번호</th>
                        <th scope="col">모델구분</th>
                        <th scope="col">제품번호</th>
                        <th scope="col">제품명</th>
                        <th scope="col">제조사</th>
                        <th scope="col">판매가격</th>
                    </tr>
                </thead>
                <tbody v-for="item in productList" :key="item.model_code">
                    <tr @click="detailview(item.model_code)">
                        <td>{{ item.model_code }}</td>
                        <td>{{ item.model_name }}</td>
                        <td>{{ item.pd_code }}</td>
                        <td>{{ item.pd_name }}</td>
                        <td>{{ item.pd_corp }}</td>
                        <td>{{ item.pd_price }}</td>
                    </tr>
                    <tr v-if="productList.length == 0">
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
                    :click-handler="searchProductList"
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
import productListModal from '@/components/cor/productListModal.vue';
import { openModal } from 'jenesius-vue-modal';
import Paginate from 'vuejs-paginate-next';

export default {
    data: function () {
        return {
            productList: [],
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
        this.searchProductList();
    },
    methods: {
        searchProductList: function (pageNum) {
            /* 1. 페이징 시 리스트 뽑아오기
             * 2. 서치바 리스트 뽑아오기
             * 2-1. 서치(장비 분류 및 검색어)
             */
            let vm = this;
            let params = new URLSearchParams();

            if (pageNum != null) this.currentPage = pageNum;
            params.append('currentPage', this.currentPage);
            params.append('pageSize', this.listCount);
            params.append('search', this.keyword);

            this.axios
                .post('/cor/productList.do', params)
                .then(function (response) {
                    console.log(response.data);

                    vm.productList = response.data.product;
                    vm.currentPage = response.data.currentPage;
                    vm.totalCnt = response.data.totalCnt;
                    vm.totalPage = vm.page();
                    vm.keyword = '';
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        },
        // 모달 상세 정보
        detailview: async function (modelCode) {
            console.log('detailview');

            const modal = await openModal(productListModal, {
                title: '제품 상세보기',
                code: modelCode,
                pageNum: this.currentPage,
                listCount: this.listCount,
            });

            modal.onclose = (popupparam) => {
                console.log('Close : ' + popupparam);
                this.searchProductList(); // 모달이 닫히면서 다시 리스트 뽑아오기
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
