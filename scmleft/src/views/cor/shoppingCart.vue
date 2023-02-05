<template>
    <div id="productList">
        <div class="content">
            <p class="Location">
                <a href="/dashboard/home" class="btn_set home">메인으로</a>
                <a href="#" class="btn_nav">주문</a>
                <span class="btn_nav bold">장바구니 목록</span>
                <a href="../system/comnCodMgr.vue" class="btn_set refresh">새로고침</a>
            </p>
            <p class="conTitle">
                <span>장바구니 목록</span>
            </p>
            <table class="col">
                <colgroup>
                    <col width="10%" />
                    <col width="30%" />
                    <col width="10%" />
                    <col width="10%" />
                    <col width="20%" />
                    <col width="20%" />
                </colgroup>
                <thead>
                    <tr>
                        <th scope="col">선택</th>
                        <th scope="col">제품명</th>
                        <th scope="col">단가</th>
                        <th scope="col">수량</th>
                        <th scope="col">합계</th>
                        <th scope="col">배송희망일</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="item in basketList" :key="item.modelCode">
                        <td>
                            <input type="checkbox" name="isCheck" />
                        </td>
                        <td>
                            <img
                                v-if="item.pdNadd == null"
                                src="/serverfile\\product\\product_none.gif" />
                            <img v-else src='"{{ item.pdNadd }}"' />
                            {{ item.pdName }}
                        </td>
                        <td>{{ comma(item.pdPrice) }}</td>
                        <td>{{ comma(item.baAmt) }}</td>
                        <td>{{ comma(item.total) }}</td>
                        <td>{{ date(item.baWishdate) }}</td>
                    </tr>
                    <tr v-if="basketList.length == 0">
                        <td colspan="6">장바구니에 담긴 제품이 없습니다.</td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div style="float: right; text-align: center; margin-bottom: 1.2rem">
            <span style="font-size: 1.5rem; font-weight: bold">장바구니 총액</span
            ><br /><br /><br />
            <span style="font-size: 1.5rem">{{ basketTotal }} 원</span>
            <hr />
            <a class="btn btn-danger mx-2" name="modal" v-on:click="delBasket()">
                <span>제품삭제</span>
            </a>
            <a class="btn btn-primary mx-2" name="modal" v-on:click="insJorderInfo()">
                <span>주문하기</span>
            </a>
        </div>
    </div>
</template>

<script>
export default {
    data: function () {
        return {
            basketList: [],
            basketTotal: '',
            productList: {}, // back용
        };
    },
    mounted() {
        /* Vue에서는 el방식이 안먹혀서 mounted에서 search 호출해서 리스트를 뿌려줘야 한다. */
        this.searchCart();
    },
    methods: {
        searchCart: function () {
            console.log('searchCart');
            let vm = this;

            this.axios
                .post('/cor/searchCart')
                .then(function (response) {
                    console.log(response);
                    console.log(response.data);

                    vm.basketList = response.data.shoppingCartList;
                    vm.basketTotal = response.data.basketTotal;
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        },
        // 제품 삭제 (다중)
        delBasket: function () {
            console.log('delBasket');

            let checkBox = document.getElementsByName('isCheck');
            let ii = -1;
            let vm = this;
            let param = new URLSearchParams();

            if (document.querySelectorAll("input[name='isCheck']:checked").length == 0)
                return this.$swal('제품을 선택해주세요.');

            for (let i = 0; i < checkBox.length; i++) {
                if (checkBox[i].checked == true) {
                    ii++;

                    this.productList[ii] = {};
                    this.productList[ii].loginId = this.basketList[i].loginId;
                    this.productList[ii].modelCode = this.basketList[i].modelCode;
                }
            }

            param.append('list', JSON.stringify(this.productList));

            this.axios
                .post('/cor/delBasketProduct_vue', param)
                .then(function (response) {
                    console.log(response);
                    console.log(response.data);

                    vm.searchCart();
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        },
        // 제품 주문 (다중)
        insJorderInfo: function () {
            console.log('insJorderInfo');

            let checkBox = document.getElementsByName('isCheck');
            let ii = -1;
            let jordIn = '0';
            let vm = this;
            let param = new URLSearchParams();

            if (document.querySelectorAll("input[name='isCheck']:checked").length == 0)
                return this.$swal('제품을 선택해주세요.');

            this.$swal
                .fire({
                    title: '입금도 같이 진행하시겠습니까?',
                    showDenyButton: true,
                    showCancelButton: false,
                    confirmButtonText: '네',
                    denyButtonText: '아니요',
                })
                .then((result) => {
                    /* Read more about isConfirmed, isDenied below */
                    if (result.isConfirmed) jordIn = '1';

                    for (let i = 0; i < checkBox.length; i++) {
                        if (checkBox[i].checked == true) {
                            ii++;

                            console.log('modelCode check :: ' + this.basketList[i].modelCode);

                            this.productList[ii] = {};
                            this.productList[ii].loginId = this.basketList[i].loginId;
                            this.productList[ii].modelCode = this.basketList[i].modelCode;
                            this.productList[ii].jordAmt = this.basketList[i].baAmt;
                            this.productList[ii].jordWishdate = this.basketList[i].baWishdate;
                            this.productList[ii].jordIn = jordIn;
                        }
                    }

                    param.append('list', JSON.stringify(this.productList));
                    console.log(this.productList);
                    console.log(JSON.stringify(this.productList));

                    this.axios
                        .post('/cor/insJorder_vue', param)
                        .then(function (response) {
                            console.log(response);
                            console.log(response.data);

                            vm.searchCart();
                        })
                        .catch(function (error) {
                            alert('에러! API 요청에 오류가 있습니다. ' + error);
                        });
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
#groupTitle {
    text-decoration: underline;
    font-weight: bold;
    cursor: pointer;
}
</style>
