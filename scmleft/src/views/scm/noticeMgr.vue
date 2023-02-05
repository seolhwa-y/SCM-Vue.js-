<template>
  <div id="comnCodMgr">
    <p class="Location">
      <a href="/dashboard/home" class="btn_set home"></a>
      <span class="btn_nav bold">기준정보</span>
      <span class="btn_nav bold">공지사항</span>
      <a href="../system/comnCodMgr.do" class="btn_set refresh">새로고침</a>
    </p>

    <p class="conTitle">
      <span>공지 사항</span>
      <span>
        <table
          style="collapse; border: 1px #50bcdf;"
          width="100%"
          cellpadding="5"
          cellspacing="0"
          border="1"
          align="left"
        >
          <tr style="border: 0px; border-color: blue">
            <td
              width="50"
              height="25"
              style="font-size: 100%; text-align: left"
            >
              <div
                id="searchArea"
                name="searchArea"
                class="d-flex justify-content-around"
              >
                <select
                  id="sea rchKey"
                  name="searchKey"
                  class="form-control"
                  style="width: 150px"
                  v-model="searchKey"
                >
                  <option value="TC">전체</option>
                  <option value="T">제목</option>
                  <option value="W">작성자</option>
                </select>

                <input
                  type="text"
                  style="width: 55%"
                  class="form-control"
                  id="searchword"
                  name="searchword"
                  v-model="searchword"
                />
                <span class="fr">
                  <a
                    class="btn btn-primary mx-2"
                    id="btnSearchGrpcod"
                    name="btnSearchGrpcod"
                    @click="search()"
                  >
                    <span>검 색</span>
                  </a>
                  <a
                    class="btn btn-primary mx-2"
                    name="modal"
                    v-on:click="getNoticeDetail()"
                  >
                    <span>신규등록</span>
                  </a>
                </span>
              </div>
            </td>
          </tr>
        </table>
      </span>
    </p>

    <div class="divComGrpCodList">
      <table class="col">
        <caption>
          caption
        </caption>
        <colgroup>
          <col width="10%" />
          <col width="10%" />
          <col width="20%" />
          <col width="30%" />
          <col width="10%" />
          <col width="10%" />
        </colgroup>

        <thead>
          <tr>
            <th scope="col">공지코드</th>
            <th scope="col">작성자ID</th>
            <th scope="col">작성자</th>
            <th scope="col">제목</th>
            <th scope="col">작성일자</th>
            <th scope="col">조회수</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="item in items"
            :key="item.notcode"
            v-on:click="getNoticeDetail(item.notCode)"
          >
            <td>{{ item.notCode }}</td>
            <td>{{ item.loginId }}</td>
            <td>{{ item.name }}</td>
            <td>{{ item.notTitle }}</td>
            <td>{{ item.notDate }}</td>
            <td>{{ item.notView }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div id="comnGrpCodPagination">
      <paginate
        class="justify-content-center"
        v-model="currentPage"
        :page-count="totalPage"
        :page-range="5"
        :margin-pages="0"
        :click-handler="search"
        :prev-text="'Prev'"
        :next-text="'Next'"
        :container-class="'pagination'"
        :page-class="'page-item'"
      >
      </paginate>
    </div>
  </div>
</template>
<script>
import { openModal } from "jenesius-vue-modal";
import noticeModal from "@/components/scm/noticeModal.vue";
import Paginate from "vuejs-paginate-next";

export default {
  data: function () {
    return {
      items: [],
      currentPage: 1,
      pageSize: 10,
      totalPage: 1,
      totalCnt: 0,
      searchKey: "TC",
      searchword: "",
      action: "",
      id: "",
    };
  },
  components: {
    paginate: Paginate,
  },
  mounted() {
    this.search();
  },
  methods: {
    // 검색
    search: function () {
      console.log("검색");

      let vm = this;

      let params = new URLSearchParams();
      params.append("pageNum", this.currentPage);
      params.append("listCount", this.pageSize);
      params.append("type", this.searchKey);
      params.append("keyword", this.searchword);

      this.axios
        .post("/scm/searchNoticeList", params)
        .then(function (response) {
          console.log(response);
          console.log(response.data);

          vm.items = response.data.result.noticeList;
          vm.totalCnt = response.data.result.noticeCount;
          vm.totalPage = vm.page();
          vm.id = response.data.loginId;
        })
        .catch(function (error) {
          alert("에러! API 요청에 오류가 있습니다. " + error);
        });
    }, // 페이지 계산
    page: function () {
      console.log("페이지");

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
    // 모달 활성화 :: 등록 및 상세보기
    getNoticeDetail: async function (notCode) {
      console.log("상세보기");

      if (notCode == null || notCode == "") {
        this.action = "I";

        const modal = await openModal(noticeModal, {
          title: "공지사항 등록",
          code: "",
          id: this.id,
          action: this.action,
        });

        modal.onclose = (popupparam) => {
          console.log("Close : " + popupparam);
          this.search();
        };
      } else {
        console.log(notCode);

        this.action = "U";
        const modal = await openModal(noticeModal, {
          title: "공지사항 상세보기",
          code: notCode,
          id: this.id,
          action: this.action,
        });

        modal.onclose = (popupparam) => {
          console.log("Close : " + popupparam);
          this.search();
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
