<template>
    <div id="productModal">
        <strong style="font-size: 30px">{{ title }}</strong>
        <table id="productDetail" class="col">
            <tbody>
                <tr>
                    <th scope="row">모델번호</th>
                    <td>{{ model_code }}</td>
                    <th scope="row">제품이미지</th>
                    <td colspan="2" style="text-align: center">
                        <img
                            id="tempImg"
                            style="object-fit: cover; max-width: 100%"
                            :src="imagepath"
                            alt="제품사진미리보기" />
                    </td>
                </tr>
                <tr>
                    <th scope="row">제조사</th>
                    <td>{{ pd_corp }}</td>
                    <th scope="row">주문수량</th>
                    <td>
                        <input type="number" min="0" v-model="jordAmt" />
                    </td>
                </tr>

                <tr>
                    <th scope="row">판매가격</th>
                    <td>{{ comma(pd_price) }}</td>

                    <th scope="row">납품희망일</th>
                    <td>
                        <input
                            type="date"
                            naem="jordWishdate"
                            v-model="jordWishdate"
                            v-bind:min="dateMin" />
                    </td>
                </tr>
                <tr>
                    <th scope="row">상세정보</th>
                    <td colspan="3">
                        <textarea
                            id="pro_det"
                            maxlength="500"
                            name="pro_det"
                            style="height: 130px; outline: none; resize: none"
                            v-model="pd_detail"
                            readonly></textarea>
                    </td>
                </tr>
            </tbody>
        </table>
        <div class="btn_areaC mt30">
            <a @click="insBasket()" class="btn btn-success">
                <span>장바구니 담기</span>
            </a>
            &nbsp;&nbsp;&nbsp;
            <a @click="insJordInfo()" class="btn btn-primary">
                <span>주문하기</span>
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
            model_code: this.code,
            pd_corp: '',
            jordAmt: '',
            pd_price: '',
            jordWishdate: '',
            pd_detail: '',
            imagepath: '',
            dateMin: new Date().toISOString().substring(0, 10),
        };
    },
    components: {},
    computed: {},
    watch: {
        // Input 숫자 아닌 문자값 입력시 대체
        number: function () {
            return (this.jordAmt = this.jordAmt.replace(/[^0-9]/g, '')); //정규식 사용
        },
    },
    // html 로딩, 가상 dom 실행, 이 두 개 연결 시 작동
    mounted: function () {
        /* modelCode 들고 가서 상세내역 가져와서 리스트에 넣어주기 */
        let vm = this;
        let param = new URLSearchParams();

        param.append('result', this.model_code);
        param.append('pageNum', 1);
        param.append('listCount', 10);

        this.axios
            .post('/cor/detailproduct.do', param)
            .then(function (response) {
                console.log(response.data);

                vm.pd_corp = response.data.result.pd_corp;
                vm.pd_price = response.data.result.pd_price;
                vm.pd_detail = response.data.result.pd_detail;
                if (response.data.result.pd_nadd != null)
                    vm.imagepath = response.data.result.pd_nadd;
                else vm.imagepath = '/serverfile\\product\\product_none.gif';
            })
            .catch(function (error) {
                alert('에러! API 요청에 오류가 있습니다. ' + error);
            });
    },
    methods: {
        // 장바구니 담기
        insBasket: function () {
            console.log('장바구니 담기');

            let vm = this;
            let param = new URLSearchParams();

            if (this.jordAmt == 0) return this.$swal('주문수량을 입력하세요.');
            if (this.jordWishdate == '') return this.$swal('주문희망일자를 선택해 주세요.');

            param.append('modelCode', this.model_code);
            param.append('bWishdate', this.jordWishdate);
            param.append('bAmt', this.jordAmt);

            this.axios
                .post('/cor/inBasket.do', param)
                .then(function (response) {
                    console.log(response.data);

                    vm.$swal(response.data.resultMsg);
                    closeModal(vm);
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        },
        // 주문하기
        insJordInfo: function () {
            console.log('주문하기');

            let vm = this;
            let param = new URLSearchParams();
            let jordIn = '0';

            if (this.jordAmt == 0) return this.$swal('주문수량을 입력하세요.');
            if (this.jordWishdate == '') return this.$swal('주문희망일자를 선택해 주세요.');

            this.$swal
                .fire({
                    title: '입금도 같이 진행하시겠습니까?',
                    showDenyButton: true,
                    showCancelButton: false,
                    confirmButtonText: '네',
                    denyButtonText: '아니요',
                })
                .then((result) => {
                    if (result.isConfirmed) jordIn = '1';

                    param.append('modelCode', this.model_code);
                    param.append('jordIn', jordIn);
                    param.append('jordWishdate', this.jordWishdate);
                    param.append('jordAmt', this.jordAmt);

                    this.axios
                        .post('/cor/inOrder.do', param)
                        .then(function (response) {
                            console.log(response.data);

                            vm.$swal(response.data.resultMsg);
                            closeModal(vm);
                        })
                        .catch(function (error) {
                            alert('에러! API 요청에 오류가 있습니다. ' + error);
                        });
                });
        },
        comma: function (num) {
            return String(num).replace(/\B(?=(\d{3})+(?!\d))/g, ',');
        },
    },
    created() {
        // 자식요소에서 부모 요소 method 호출
    },
};
</script>

<style>
#productDetail {
    width: 66rem;
    margin: 2rem;
    font-size: 1rem;
    height: 6rem;
}
#productModal {
    background-color: #fff;
    padding: 3rem;
    border-radius: 10px;
    border: 2px solid rgb(59, 59, 59);
    text-align: center;
}
</style>
