<template>
  <div>
    <div>
      <h1>구독 변경 신청</h1>
      <br />
      <div id="inner">
        <p class="tip">서로 다른 정책의 제품들의 일괄 구매를 원하시는 고객님께서는 견적 문의 부탁드립니다.</p>
        <br />
        <form class="form">
          <table class="table_add">
            <tr>
              <th>현재 구독정책명</th>
              <td>{{policy_title}}</td>
            </tr>
            <tr>
              <th>변경할 구독 선택</th>
              <td>
                <select
                  v-model="standardBoth"
                  class="select-box"
                  style="width:400px"
                  @change="changeSubList"
                >
                  <option disabled value>이용기준을 선택해주세요</option>
                  <option value="U">사용자</option>
                  <option value="A">에이전트</option>
                </select>
                <br />
                <select v-model="selected" class="select-box" style="width:400px">
                  <option disabled value>정책을 선택해주세요</option>
                  <option v-for="list in policyList" :value="list.no" :key="list.no">
                    {{list.policy_title}}&nbsp;&nbsp; [ ￦{{list.price | formatPrice}} ]
                    사용수 : {{list.max_count}}
                  </option>
                </select>
              </td>
            </tr>
            <tr>
              <th>구독 이용 시작일</th>
              <td>
                <input type="date" v-model="startdate" /> &nbsp;
              </td>
            </tr>
            <tr v-if="startdate!='' && usemonth!=''">
              <th>구독 이용 종료일</th>
              <td
                style="font-weight: normal; letter-spacing: 1px;"
              >{{ getEndDate=='Invalid date' ? '' : getEndDate }}</td>
            </tr>
            <tr>
              <th>이용 개월 수</th>
              <td>
                <input type="number" min="1" max="36" v-model="usemonth" @change="usemonthCheck" /> 개월
                <div style="font-size:12px; color:red;">* 개월 수는 30일로 계산됩니다.</div>
              </td>
            </tr>
          </table>
          <br />

          <div class="submit_btn">
            <button type="button" @click="listClick">취소</button>
            <button type="button" @click="changeSubmit">신청</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import moment from "moment";
import { subscribeAllList_P, subscribeChange } from "@/api/shr/subscribe";
export default {
  data() {
    return {
      license_no: "",
      policy_no: "",
      policy_title: "",
      startdate: "",
      enddate: "",
      usemonth: "",
      policyList: "",
      policyAllList: "",
      selected: "",
      standardBoth: ""
    };
  },
  async created() {
    this.license_no = this.$route.params.license_no;
    this.policy_title = this.$route.params.policy_title;
    const { data } = await subscribeAllList_P();
    this.policyAllList = data;
  },
  computed: {
    getEndDate: function() {
      const enddatee = moment(this.startdate)
        .add(this.usemonth * 30, "d")
        .format("YYYY-MM-DD");
      return enddatee;
    }
  },

  methods: {
    // 목록
    listClick() {
      this.$router.push({ name: "licneseList" });
    },
    // 구독 변경 신청
    async changeSubmit() {
      // 유효성 검사
      if (
        this.usemonth == "" ||
        this.startdate == "" ||
        this.getEndDate == "" ||
        this.selected == "" ||
        this.standardBoth == ""
      ) {
        alert("필수 입력사항을 입력해주세요");
      } else {
        const changeData = {
          license_no: this.license_no,
          policy_no: this.selected,
          month_count: this.usemonth,
          start_date: this.startdate,
          end_date: this.getEndDate
        };

        const { data } = await subscribeChange(changeData);
        if (data == 1) {
          alert("변경 신청이 완료되었습니다.");
          this.$router.push("/license/list");
        } else {
          alert("변경 신청 실패");
        }
      }
    },
    // 이용기준 선택에 따른 정책 출력
    changeSubList() {
      if (this.standardBoth == "A") {
        const x = this.policyAllList.filter(x => {
          return x.standard == "A";
        });
        this.policyList = x;
      } else if (this.standardBoth == "U") {
        const x = this.policyAllList.filter(x => {
          return x.standard == "U";
        });
        this.policyList = x;
      }
    },
    // usemonth 체크
    usemonthCheck() {
      if (!(this.usemonth >= 1 && this.usemonth <= 36)) {
        alert("1개월 이상 최대 36개월까지 가능합니다.");
        this.usemonth = 1;
      }
    }
  }
};
</script>


<style scoped>
h1 {
    font-size: 4.5rem;
    color:#000000D9;
}
.tip {
  border: 2px solid gray;
  color: red;
  font-size: 15px;
  padding: 20px;
  border-radius: 5px;
}
button {
  width: auto;
  padding: 0 15px;
  margin: 10px;
}
input[type="date"] {
  text-align: center;
  border-style: none;
  font-size: 100%;
  border: 1px solid #dae1e7;
  -webkit-box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.1);
  box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.1);
  padding: 0.5rem 0.75rem;
  margin: 5px;
   outline:none;
}
#inner {
  width: 100%;
  display: inline-block;
  border: 1px solid #ccc;
  padding:10px;
}
</style>