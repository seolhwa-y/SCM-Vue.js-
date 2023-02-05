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
                            <span class="btn_nav bold">창고 정보 관리</span>
                            <a
                                onClick="top.location='javascript:location.reload()'"
                                class="btn_set refresh"
                                >새로고침</a
                            >
                        </p>

                        <p class="conTitle">
                            <span>창고 정보 관리 </span>
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
                                            id="selectS"
                                            name="selectS"
                                            style="width: 130px; height: 27px"
                                            v-model="selectS">
                                            <option value="all">전체</option>
                                            <option value="wh_name">창고명</option>
                                            <option value="wh_addr">창고위치</option>
                                        </select>

                                        <input
                                            type="text"
                                            style="width: 150px; height: 25px"
                                            id="search"
                                            name="search"
                                            v-model="search" />
                                        <a
                                            @click="getList()"
                                            class="btnType blue"
                                            id="serachbtn"
                                            name="btn"
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
                                        <th scope="col">창고코드</th>
                                        <th scope="col">창고명</th>
                                        <th scope="col">우편번호</th>
                                        <th scope="col">창고위치</th>
                                        <th scope="col">상세주소</th>
                                        <th scope="col">담당자</th>
                                    </tr>
                                </thead>
                                <!--기존 자바스크립트 방법  <tbody id="productList"></tbody> -->
                                <tbody v-if="salesList.length == 0">
                                    <tr>
                                        <td colspan="7" class="text-center">
                                            등록된 창고가 없습니다
                                        </td>
                                    </tr>
                                </tbody>
                                <tbody v-else v-for="item in salesList" :key="item.wh_CODE">
                                    <tr @click="openpop(item.wh_CODE, item.loginID, item.name)">
                                        <td>{{ item.wh_CODE }}</td>
                                        <td>{{ item.wh_NAME }}</td>
                                        <td>{{ item.wh_ZIP }}</td>
                                        <td>{{ item.wh_ADDR }}</td>
                                        <td>{{ item.wh_ADDR_DTL }}</td>
                                        <td>{{ item.name }}</td>
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
                                :click-handler="getList"
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
import Paginate from 'vuejs-paginate-next';
import { openModal } from 'jenesius-vue-modal';
import whModal from './whmodal.vue';

export default {
    data: () => {
        return {
            currentPage: 1,
            pageSize: 5,
            totalCount: 0,
            totalPage: 1,
            search: '',
            selectS: 'all',
            action: '',

            salesList: [],
        };
    },
    components: { paginate: Paginate },
    mounted: function () {
        this.getList();
    },
    methods: {
        getList: function () {
            let vm = this;

            let params = new URLSearchParams();
            params.append('currentPage', this.currentPage);
            params.append('pageSize', this.pageSize);
            params.append('search', this.search);
            params.append('selectS', this.selectS);

            this.axios
                .post('/scm/listWhvue.do', params)
                .then((resp) => {
                    console.log(resp);
                    let data = resp.data;

                    vm.totalCount = data.WhCnt;
                    vm.totalPage = vm.page();
                    vm.salesList = data.WhList;
                })
                .catch((error) => {
                    console.log(error);
                });
        },
        page: function () {
            var total = this.totalCount;
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
        openpop: async function (whno, loginID, name) {
            if (whno == '' || whno == null) {
                this.action = 'I';
                const modal = await openModal(whModal, {
                    whno: 0,
                    name: '',
                    loginID: '',
                    action: this.action,
                });
                console.log('asd : ' + this.action);
                modal.onclose = () => {
                    this.getList();
                };
            } else {
                this.action = 'U';
                const modal = await openModal(whModal, {
                    whno: whno,
                    name: name,
                    loginID: loginID,
                    action: this.action,
                });
                modal.onclose = () => {
                    this.getList();
                };
            }
        },
    },
};
</script>

<style>
.searchArea {
    margin-top: 35px;
    padding: 50px 0;
    border: 2px solid rgb(190, 190, 190);
}
#searchBtnWrap {
    display: inline-block;
    margin: 0 10px;
}
#whListArea {
    margin-bottom: 30px;
}
#whListArea table,
#whInfoArea table {
    margin: 0 auto;
}

#btn-close-daum {
    position: absolute;
    right: 0;
    bottom: 0;
    z-index: 11;
    cursor: pointer;
}

#formwrap {
    margin-top: 50px;
    margin-bottom: 50px;
    border: 2px solid rgb(190, 190, 190);
    padding-left: 50px;
}
#formwrap #formtable {
    border-collapse: separate;
    border-spacing: 10px 10px;
    margin: 10px auto;
}
#formwrap #phoneArea input {
    width: 100px;
}
</style>
