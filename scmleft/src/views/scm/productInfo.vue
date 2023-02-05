<!-- eslint-disable prettier/prettier -->
<template>
    <div id="wrap_area">
        <h2 class="hidden">컨텐츠 영역</h2>
        <div id="container">
            <ul>
                <li class="contents">
                    <!-- contents -->
                    <h3 class="hidden">contents 영역</h3>
                    <!-- content -->
                    <div class="content">
                        <p class="Location">
                            <a href="/dashboard/dashboard.do" class="btn_set home">메인으로</a>
                            <a href="#" class="btn_nav">기준 정보</a>
                            <span class="btn_nav bold">제품정보 관리</span>
                            <a
                                onClick="top.location='javascript:location.reload()'"
                                class="btn_set refresh"
                                >새로고침</a
                            >
                        </p>

                        <p class="conTitle">
                            <span>제품정보 관리 </span>
                            <span class="fr">
                                <a class="btnType blue" name="modal" @click="openpop()">
                                    <span>신규등록</span></a
                                >
                            </span>
                        </p>

                        <!--검색창   -->
                        <div id="searchproduct">
                            <table
                                width="100%"
                                cellpadding="5"
                                cellspacing="0"
                                border="1"
                                align="left"
                                style="border-collapse: collapse; border: 1px #50bcdf">
                                <tr style="border: 0px; border-color: blue">
                                    <td width="100" height="25" style="font-size: 120%">
                                        &nbsp;&nbsp;
                                    </td>
                                    <td
                                        width="50"
                                        height="25"
                                        style="
                                            font-size: 100%;
                                            text-align: right;
                                            padding-right: 25px;
                                        ">
                                        <select
                                            id="select"
                                            name="select"
                                            style="width: 130px; height: 27px"
                                            v-model="select">
                                            <option value="all">전체</option>
                                            <option value="pro_no">제품 코드</option>
                                            <option value="pro_nm">제품명</option>
                                            <option value="model_nm">모델명</option>
                                            <option value="pd_corp">제조사</option>
                                        </select>

                                        <input
                                            type="text"
                                            style="width: 150px; height: 25px"
                                            id="search"
                                            name="search"
                                            v-model="search" />
                                        <a href="" class="btnType blue" id="serachbtn" name="btn"
                                            ><span>검 색</span></a
                                        >
                                    </td>
                                </tr>
                            </table>
                            &nbsp;&nbsp;
                        </div>

                        <div id="productList">
                            <table class="col">
                                <caption>
                                    caption
                                </caption>

                                <thead>
                                    <tr>
                                        <th scope="col"></th>
                                        <th scope="col">모델명</th>
                                        <th scope="col">제품 코드</th>
                                        <th scope="col">제품명</th>
                                        <th scope="col">제조사</th>
                                        <th scope="col">판매가</th>
                                        <th scope="col">납품 업체</th>
                                    </tr>
                                </thead>
                                <!--기존 자바스크립트 방법  <tbody id="productList"></tbody> -->
                                <tbody>
                                    <tr v-for="item in listitem" :key="item.modelCode">
                                        <td>{{ item.modelCode }}</td>
                                        <td>
                                            <a @click="openpop(item.modelCode)">{{
                                                item.modelName
                                            }}</a>
                                        </td>
                                        <td>{{ item.pdCode }}</td>
                                        <td>{{ item.pdName }}</td>
                                        <td>{{ item.pdCorp }}</td>
                                        <td>{{ item.pdPrice }}</td>
                                        <td>{{ item.company }}</td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                        <div id="productPagination">
                            <paginate
                                class="justify-content-center"
                                v-model="currentPage"
                                :page-count="totalPage"
                                :page-range="5"
                                :margin-pages="0"
                                :click-handler="listsearch"
                                :prev-text="'Prev'"
                                :next-text="'Next'"
                                :container-class="'pagination'"
                                :page-class="'page-item'">
                            </paginate>
                        </div>
                    </div>
                    <!--// content -->
                </li>
            </ul>
        </div>
    </div>
</template>
<script>
import { openModal } from 'jenesius-vue-modal';
import productModal from '@/components/scm/productModal.vue';
import Paginate from 'vuejs-paginate-next';

export default {
    data: function () {
        return {
            listitem: [], // 변수선언 ex.<tbody v-for="(item, index) in SupplierListItem" v-if="SupplierListItem.length">
            action: '',
            currentPage: 1,
            pageSize: 10,
            totalPage: 1,
            totalCnt: 0,
            search: '',
            select: 'all',
        };
    },
    components: {
        paginate: Paginate,
    },
    mounted() {
        this.listsearch();
    },
    methods: {
        listsearch: function () {
            let vm = this;

            let params = new URLSearchParams();
            params.append('currentPage', this.currentPage);
            params.append('pageSize', this.pageSize);
            params.append('search', this.search);
            params.append('select', this.select);

            this.axios
                .post('/scm/productInfoList.do', params)
                .then(function (response) {
                    console.log(response);
                    vm.listitem = response.data.productInfoList;
                    vm.totalCnt = response.data.totalCnt;
                    vm.totalPage = vm.page();
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        },
        page: function () {
            var total = this.totalCnt;
            var page = this.pageSize;
            var xx = total % page;
            var result = parseInt(total / page);

            if (xx == 0) {
                return result;
            } else {
                result = result + 1;
                return result;
            }
        },
        openpop: async function (selmodelCode) {
            if (selmodelCode == '' || selmodelCode == null) {
                alert('등록');
                this.action = 'I';
                const modal = await openModal(productModal, {
                    ptitle: '제품 등록',
                    selmodelCode: 0,
                    action: this.action,
                });

                modal.onclose = () => {
                    console.log('Close ');
                    this.listsearch();
                };
            } else {
                this.action = 'U';
                const modal = await openModal(productModal, {
                    ptitle: '제품 수정',
                    selmodelCode: selmodelCode,
                    action: this.action,
                });

                modal.onclose = () => {
                    console.log('Close ');
                    this.listsearch();
                };
            }
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
