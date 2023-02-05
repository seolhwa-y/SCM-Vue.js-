<template>
    <form id="proctioninfo">
        <dl id="grpInfoWrap">
            <dd class="content">
                <!-- s : 여기에 내용입력 -->

                <table id="grpInfo" width="800px">
                    <caption>
                        caption
                    </caption>
                    <colgroup>
                        <col width="120px" />
                        <col width="*" />
                        <col width="120px" />
                        <col width="*" />
                    </colgroup>

                    <tbody>
                        <tr>
                            <td colspan="4" class="text-center">
                                <div class="my-4">
                                    <strong style="font-size: 30px">{{ ptitle }}</strong>
                                </div>
                            </td>
                        </tr>
                        <tr>
                            <th scope="row">제품 코드 <span class="font_red">*</span></th>
                            <td colspan="3">
                                <input
                                    class="ui-widget ui-widget-content ui-corner-all"
                                    style="height: 20px"
                                    type="text"
                                    maxlength="15"
                                    name="pdCode"
                                    id="pdCode"
                                    v-model="modelCode" />
                            </td>
                        </tr>

                        <tr>
                            <th scope="row">제품명 <span class="font_red">*</span></th>
                            <td>
                                <input
                                    class="ui-widget ui-widget-content ui-corner-all"
                                    style="height: 20px"
                                    type="text"
                                    maxlength="15"
                                    name="pdName"
                                    id="pdName"
                                    v-model="pdName" />
                                <SelectComCombo
                                    comtype="wh"
                                    selectid="whcode"
                                    type="sel"
                                    selvalue=""
                                    multiple />
                            </td>

                            <th scope="row">모델명 <span class="font_red">*</span></th>
                            <td>
                                <input
                                    class="ui-widget ui-widget-content ui-corner-all"
                                    style="height: 20px"
                                    type="text"
                                    maxlength="15"
                                    name="modelName"
                                    id="modelName"
                                    v-model="modelName" />
                            </td>
                        </tr>
                        <tr>
                            <th scope="row">제조사 <span class="font_red">*</span></th>
                            <td>
                                <input
                                    class="ui-widget ui-widget-content ui-corner-all"
                                    style="height: 20px"
                                    type="text"
                                    maxlength="15"
                                    name="pdCorp"
                                    id="pdCorp"
                                    v-model="pdCorp" />
                            </td>

                            <th scope="row">제품 가격 <span class="font_red">*</span></th>
                            <td>
                                <input
                                    class="ui-widget ui-widget-content ui-corner-all"
                                    style="height: 20px"
                                    type="text"
                                    maxlength="10"
                                    name="pdPrice"
                                    id="pdPrice"
                                    v-model="pdPrice" />
                            </td>
                        </tr>
                        <tr>
                            <th scope="row">납품 업체 <span class="font_red">*</span></th>
                            <td>
                                <Usercombo
                                    usertype="F"
                                    selectid="loginId"
                                    type="sel"
                                    selvalue=""
                                    v-model="loginId" />
                            </td>
                        </tr>

                        <tr>
                            <th scope="row">상세 정보 <span class="font_red">*</span></th>
                            <td colspan="3">
                                <textarea
                                    class="ui-widget ui-widget-content ui-corner-all"
                                    id="pdDetail"
                                    maxlength="500"
                                    name="pdDetail"
                                    style="height: 130px; outline: none; resize: none"
                                    placeholder="여기에 상세정보를 적어주세요.(500자 이내)"
                                    v-model="pdDetail"></textarea>
                            </td>
                        </tr>

                        <!-- 첨부파일&다운로드  -->
                        <tr>
                            <th scope="row">대표 이미지 <span class="font_red">*</span></th>
                            <td class="thumb">
                                <span>
                                    <input
                                        name="thumbnail"
                                        type="file"
                                        id="thumbnail"
                                        @change="selimg" />
                                </span>
                            </td>
                            <td colspan="2" style="text-align: center">
                                <img
                                    id="tempImg"
                                    style="object-fit: cover; max-width: 100%"
                                    alt="제품사진미리보기"
                                    v-bind:src="pdNadd"
                                    @click="download()" />
                            </td>
                        </tr>
                    </tbody>
                </table>

                <!-- e : 여기에 내용입력 -->

                <div class="btn_areaC mt30">
                    <a @click="save()" class="btnType blue" id="btnSavePro" name="btn"
                        ><span>저장</span></a
                    >
                    <a @click="closepopup()" class="btnType gray" id="btnClose" name="btn"
                        ><span>취소</span></a
                    >
                </div>
            </dd>
        </dl>
    </form>
</template>

<script>
import { closeModal } from 'jenesius-vue-modal';
import Usercombo from '@/components/common/Usercombo.vue';
import SelectComCombo from '@/components/common/SelectComCombo.vue';

export default {
    // vue에서는 받아온 변수를 methods에서 직접 핸들링이 불가능하기 때문에
    // 임시 변수를 만들어서 받아온 변수를 넣어 줘야 함
    props: { ptitle: String, selmodelCode: Number, action: String },
    data: function () {
        return {
            pmodelCode: this.selmodelCode,
            modelCode: '',
            pdCode: '',
            pdName: '',
            modelName: '',
            pdCorp: '',
            pdPrice: '',
            company: '',
            pdDetail: '',
            loginId: '',
            pdfileCode: '',
            pdfileName: '',
            pdNadd: '/images/admin/comm/no_image.png',
            pdMadd: '',
            pdSize: '',
            paction: this.action,
        };
    },
    components: {
        Usercombo: Usercombo,
        SelectComCombo: SelectComCombo,
    },
    computed: {},
    // html 로딩, 가상 dom 실행, 이 두 개 연결 시 작동
    mounted: function () {
        let vm = this;
        // 신규 등록 시
        if (this.pmodelCode == null || this.pmodelCode == '') {
            vm.modelCode = '';
            vm.pdCode = '';
            vm.pdName = '';
            vm.modelName = '';
            vm.pdCorp = '';
            vm.pdPrice = '';
            vm.company = '';
            vm.pdDetail = '';
            vm.loginId = '';
            vm.pdfileCode = '';
            vm.pdfileName = '';
            vm.pdNadd = '/images/admin/comm/no_image.png';
            vm.pdMadd = '';
            vm.pdSize = '';
        } else {
            //  수정 시 (grp_cod 에 해당하는 상세코드 정보 가져오기)
            let params = new URLSearchParams();
            params.append('modelCode', this.pmodelCode);

            this.axios
                .post('/scm/productOne.do', params)
                .then(function (response) {
                    console.log(response);

                    //vm.title = response.data.selectresult.nottitle;
                    //vm.cont = response.data.selectresult.notcon;
                    vm.modelCode = response.data.result.modelCode;
                    vm.pdCode = response.data.result.pdCode;
                    vm.pdName = response.data.result.pdName;
                    vm.modelName = response.data.result.modelName;
                    vm.pdCorp = response.data.result.pdCorp;
                    vm.pdPrice = response.data.result.pdPrice;
                    vm.company = response.data.result.company;
                    vm.pdDetail = response.data.result.pdDetail;
                    vm.loginId = response.data.result.loginId;
                    vm.pdfileCode = response.data.result.pdfileCode;
                    vm.pdfileName = response.data.result.pdfileName;
                    vm.pdNadd = response.data.result.pdNadd;
                    vm.pdMadd = response.data.result.pdMadd;
                    vm.pdSize = response.data.result.pdSize;
                })
                .catch(function (error) {
                    alert('에러! API 요청에 오류가 있습니다. ' + error);
                });
        }
    },
    methods: {
        selimg: function () {
            var image = event.target;

            if (image.files[0]) {
                this.pdNadd = window.URL.createObjectURL(image.files[0]);
            }
        },
        save: function () {
            //alert(this.loginId + ' : ' + this.whcode);
            if (confirm('저장하시겠습니까?')) {
                //let vm = this;
                //let params = new URLSearchParams();
                //params.append('title', this.title);
                //params.append('cont', this.cont);
                //params.append('noticeno', this.noticeno);
                //params.append('action', this.paction);

                let params = new FormData(document.getElementById('proctioninfo'));
                params.append('action', this.paction);
                params.append('modelCode', this.pmodelCode);

                this.axios(
                    {
                        url: '/scm/productSave.do',
                        method: 'POST',
                        data: params,
                    },
                    {
                        headers: {
                            'Content-Type': 'multipart/form-data',
                        },
                    }
                )
                    .then(function (response) {
                        console.log(response);
                        let status = response.status;
                        let msg = response.data.result;
                        if (status == 200) {
                            alert('저장이 완료 되었습니다.');
                            closeModal();
                        } else {
                            alert(msg);
                        }
                    })
                    .catch(function (error) {
                        alert('에러! API 요청에 오류가 있습니다. ' + error);
                    });
            }
        },
        closepopup: function () {
            closeModal();
        },
        download: function () {
            alert('download');

            let params = new URLSearchParams();
            params.append('modelCode', this.pmodelCode);

            this.axios({
                url: '/scm/fileDown.do', // File URL Goes Here
                data: params,
                method: 'POST',
                responseType: 'blob',
            }).then((res) => {
                console.log(res);
                var FILE = window.URL.createObjectURL(new Blob([res.data]));
                console.log('FILE : ' + FILE);
                var docUrl = document.createElement('a');
                docUrl.href = FILE;
                docUrl.setAttribute('download', this.pdfileName);
                document.body.appendChild(docUrl);
                docUrl.click();
            });
        },
    },
    created() {
        // 자식요소에서 부모 요소 method 호출
    },
};
</script>

<style>
#grpInfo {
    border-collapse: separate;
    border-spacing: 20px;
}
#grpInfoWrap {
    background-color: #fff;
    padding: 3rem;
    border-radius: 10px;
    border: 2px solid rgb(59, 59, 59);
}
</style>
