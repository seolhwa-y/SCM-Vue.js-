<template>
    <div id="orderReturn">
        <div class="content">
            <p class="Location">
                <a href="#" class="btn_set home">메인으로</a>
                <a href="#" class="btn_nav">제품 발주/반품</a>
                <span class="btn_nav bold">발주/반품 처리</span>
                <a onClick="../system/comnCodMgr.vue" class="btn_set refresh">새로고침</a>
            </p>

            <p class="conTitle" id="searcharea">
                <span>발주지시서 목록</span>
                <span id="search" class="fr">
                    <select id="searchKey" name="searchKey" v-model="type">
                        <option value="" selected="selected" disabled="disabled">검색조건</option>
                        <option value="companyName">업체명</option>
                    </select>
                    <input
                        type="text"
                        id="searchWord"
                        v-model="keyword"
                        placeholder=""
                        style="height: 27px" />
                    <input type="date" v-model="startDate" />
                    <input type="date" v-model="endDate" />
                    <a class="btn btn-primary mx-2" v-on:click="searchOrderReturnList()">
                        <span>검색</span>
                    </a>
                </span>
            </p>

            <table class="col">
                <colgroup>
                    <col width="10%" />
                    <col width="30%" />
                    <col width="30%" />
                </colgroup>
                <thead>
                    <tr>
                        <th scope="col">NO</th>
                        <th scope="col">납품업체명</th>
                        <th scope="col">총액</th>
                    </tr>
                </thead>
                <tbody>
                    <tr
                        v-for="(item, index) in orderReturnList"
                        :key="item.loginId"
                        v-on:click="searchOrderReturnDetailList(1, item.loginId)">
                        <td>{{ index + 1 }}</td>
                        <td>{{ item.companyName }}</td>
                        <td>{{ item.total }}</td>
                    </tr>
                    <tr v-if="orderReturnList.length == 0">
                        <td colspan="3">검색된 데이터가 없습니다.</td>
                    </tr>
                </tbody>
            </table>

            <div id="comnGrpCodPagination">
                <paginate
                    class="justify-content-center"
                    v-model="currentPage"
                    :page-count="totalPage"
                    :page-range="5"
                    :margin-pages="0"
                    :click-handler="searchOrderReturnList"
                    :prev-text="'Prev'"
                    :next-text="'Next'"
                    :container-class="'pagination'"
                    :page-class="'page-item'">
                </paginate>
            </div>

            <!-- 발주지시서 상세내역 영역 -->
            <p class="conTitle" id="searcharea">
                <span>발주지시서 상세내역 목록</span>
            </p>

            <!-- 발주지시서 상세내역 영역 -->
            <table class="col">
                <colgroup>
                    <col width="5%" />
                    <col width="10%" />
                    <col width="15%" />
                    <col width="10%" />
                    <col width="10%" />
                    <col width="10%" />
                    <col width="10%" />
                    <col width="5%" />
                    <col width="10%" />
                    <col width="15%" />
                </colgroup>
                <thead>
                    <tr>
                        <th scope="col">체크</th>
                        <th scope="col">제품코드</th>
                        <th scope="col">제품명</th>
                        <th scope="col">제품CODE</th>
                        <th scope="col">제조사</th>
                        <th scope="col">날짜</th>
                        <th scope="col">수량</th>
                        <th scope="col">판매가격</th>
                        <th scope="col">창고이름</th>
                        <th scope="col">상태</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="item in orderReturnDetailList" :key="item.bordCode">
                        <td>
                            <input type="checkBox" name="isCheck" />
                            <!-- 반품처리 및 재고처리에 필요한 체크박스 -->
                        </td>
                        <td>{{ item.modelName }}</td>
                        <td>{{ item.pdName }}</td>
                        <td>{{ item.pdCode }}</td>
                        <td>{{ item.pdCorp }}</td>
                        <td>{{ item.dirDate }}</td>
                        <td>{{ item.bordAmt }}</td>
                        <td>{{ item.pdPrice }}</td>
                        <td>{{ item.whName }}</td>
                        <td
                            v-if="item.bordDate != null && item.reCode == 0"
                            style="color: red; font-weight: bold">
                            반품가능
                        </td>
                        <td
                            v-else-if="item.bordDate == null && item.reCode == 0"
                            style="color: blue; font-weight: bold">
                            입고가능
                        </td>
                        <td v-else-if="item.reCode != 0" style="color: green; font-weight: bold">
                            출고가능
                        </td>
                    </tr>
                </tbody>
            </table>

            <div id="comnGrpCodPagination2">
                <paginate
                    class="justify-content-center"
                    v-model="currentPage2"
                    :page-count="totalPage2"
                    :page-range="5"
                    :margin-pages="0"
                    :click-handler="searchOrderReturnDetailList"
                    :prev-text="'Prev'"
                    :next-text="'Next'"
                    :container-class="'pagination'"
                    :page-class="'page-item'">
                </paginate>
            </div>
            <br />
            <div style="margin-left: 40%">
                <a class="btn btn-danger mx-2" v-on:click="insertOrderReturn()"
                    ><span>반품처리</span></a
                >
                <a class="btn btn-primary mx-2" v-on:click="updateOrderReturn()"
                    ><span>재고처리</span></a
                >
            </div>
        </div>
    </div>
</template>
<script>
import Paginate from 'vuejs-paginate-next'; // 페이징 처리

export default {
    data: function () {
        return {
            orderReturnList: [],
            orderReturnDetailList: [],
            ormList: {},
            type: '',
            keyword: '',
            startDate: '',
            endDate: '',
            loginId: '',
            currentPage: 1, //  현재 페이지
            currentPage2: 1, //  현재 페이지
            listCount: 5, // 리스트 뿌릴 사이즈
            listCount2: 5, // 리스트 뿌릴 사이즈
            //  pageCount: 10, // 페이징 사이즈
            totalPage: 1, // 페이징
            totalPage2: 1, // 페이징
            totalCnt: 0, // 리스트의 총 사이즈
            totalCnt2: 0, // 리스트의 총 사이즈
        };
    },
    components: {
        paginate: Paginate,
    },
    mounted() {
        /* Vue에서는 el방식이 안먹혀서 mounted에서 search 호출해서 리스트를 뿌려줘야 한다. */
        this.searchOrderReturnList();
    },
    methods: {
        searchOrderReturnList: function (pageNum) {
            console.log('searchOrderReturnList');

            let vm = this;
            let params = new URLSearchParams();

            if (pageNum != null) this.currentPage = pageNum;

            params.append('pageNum', this.currentPage);
            params.append('listCount', this.listCount);
            params.append('type', this.type);
            params.append('keyword', this.keyword);
            params.append('startDate', this.startDate.replaceAll('-', ''));
            params.append('endDate', this.endDate.replaceAll('-', ''));

            this.axios
                .post('/ship/searchOrderReturn', params)
                .then(function (response) {
                    console.log(response.data);

                    vm.orderReturnDetailList = [];
                    vm.orderReturnList = response.data.result.orderReturnList;
                    vm.currentPage = parseInt(response.data.result.pageNum);
                    vm.totalCnt = response.data.result.orderReturnCount;
                    vm.totalPage = vm.page();
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        },
        // 발주지시서 상세내역 불러오기
        searchOrderReturnDetailList: function (pageNum, loginId) {
            console.log('orderReturnDetailList');

            let vm = this;
            let params = new URLSearchParams();

            if (pageNum != null) this.currentPage2 = pageNum;
            if (loginId != null) this.loginId = loginId;

            params.append('pageNum', this.currentPage2);
            params.append('listCount', this.listCount2);
            params.append('loginId', this.loginId);

            this.axios
                .post('/ship/searchOrderReturnDetail', params)
                .then(function (response) {
                    console.log(response.data);

                    vm.orderReturnDetailList = response.data.result.orderReturnDetailList;
                    vm.currentPage2 = parseInt(response.data.result.pageNum);
                    vm.totalCnt2 = response.data.result.orderReturnDetailCount;
                    vm.totalPage2 = vm.page2();

                    console.log(vm.currentPage2);
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        },
        // 반품처리
        insertOrderReturn: function () {
            console.log('insertOrderReturn');

            let ii = -1;
            let vm = this;
            let params = new URLSearchParams();
            let checkBox = document.getElementsByName('isCheck');

            for (let i = 0; i < checkBox.length; i++) {
                if (checkBox[i].checked == true) {
                    if (
                        this.orderReturnDetailList[i].bordDate != null &&
                        this.orderReturnDetailList[i].reCode == 0
                    ) {
                        ii++;

                        this.ormList[ii] = {};
                        this.ormList[ii].bordCode = this.orderReturnDetailList[i].bordCode;
                        this.ormList[ii].modelCode = this.orderReturnDetailList[i].modelCode;
                        this.ormList[ii].whCode = this.orderReturnDetailList[i].whCode;
                        this.ormList[ii].reCode = this.orderReturnDetailList[i].reCode;
                        this.ormList[ii].bordAmt = parseInt(
                            this.orderReturnDetailList[i].bordAmt.replaceAll(',', '')
                        );
                        this.ormList[ii].type = '0';
                    }
                }
            }

            params.append('ormList', JSON.stringify(this.ormList));

            this.axios
                .post('/ship/insertOrderReturn', params)
                .then(function (response) {
                    console.log(response.data);

                    for (let i = 0; i < checkBox.length; i++) {
                        if (checkBox[i].checked == true) checkBox[i].checked = false;
                    }
                    vm.ormList = {};
                    vm.searchOrderReturnDetailList();
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        },
        // 재고처리 (출고 입고)
        updateOrderReturn: function () {
            console.log('updateOrderReturn');

            let ii = -1;
            let vm = this;
            let params = new URLSearchParams();
            let checkBox = document.getElementsByName('isCheck');

            for (let i = 0; i < checkBox.length; i++) {
                if (checkBox[i].checked == true) {
                    if (
                        this.orderReturnDetailList[i].bordDate == null &&
                        this.orderReturnDetailList[i].reCode == 0
                    ) {
                        console.log('입고' + this.orderReturnDetailList[i].bordCode);
                        // 입고
                        ii++;

                        this.ormList[ii] = {};
                        this.ormList[ii].bordCode = this.orderReturnDetailList[i].bordCode;
                        this.ormList[ii].modelCode = this.orderReturnDetailList[i].modelCode;
                        this.ormList[ii].whCode = this.orderReturnDetailList[i].whCode;
                        this.ormList[ii].reCode = this.orderReturnDetailList[i].reCode;
                        this.ormList[ii].bordAmt = parseInt(
                            this.orderReturnDetailList[i].bordAmt.replaceAll(',', '')
                        );
                        this.ormList[ii].loginId = this.loginId;
                        this.ormList[ii].type = 'in';
                    } else if (
                        this.orderReturnDetailList[i].bordDate != null &&
                        this.orderReturnDetailList[i].reCode != 0
                    ) {
                        console.log('출고' + this.orderReturnDetailList[i].reCode);
                        // 출고
                        ii++;

                        this.ormList[ii] = {};
                        this.ormList[ii].bordCode = this.orderReturnDetailList[i].bordCode;
                        this.ormList[ii].modelCode = this.orderReturnDetailList[i].modelCode;
                        this.ormList[ii].whCode = this.orderReturnDetailList[i].whCode;
                        this.ormList[ii].reCode = this.orderReturnDetailList[i].reCode;
                        this.ormList[ii].bordAmt = parseInt(
                            this.orderReturnDetailList[i].bordAmt.replaceAll(',', '')
                        );
                        this.ormList[ii].loginId = this.loginId;
                        this.ormList[ii].type = 'out';
                    }
                }
            }

            params.append('ormList', JSON.stringify(this.ormList));

            this.axios
                .post('/ship/updateOrderReturn', params)
                .then(function (response) {
                    console.log(response.data);

                    for (let i = 0; i < checkBox.length; i++) {
                        if (checkBox[i].checked == true) checkBox[i].checked = false;
                    }

                    vm.ormList = {};
                    vm.searchOrderReturnDetailList();
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
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
        page2: function () {
            var total = this.totalCnt2;
            var page = this.listCount2;
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
