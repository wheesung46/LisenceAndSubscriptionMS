<template>
  <div id="order" class="contents">
    <h1>구독 갱신</h1>
    <p class="tip">
      서로 다른 정책의 제품들의 일괄 구매를 원하시는 고객님께서는 견적 문의
      부탁드립니다.
    </p>
    <div class="info-border">
      <template>
        <table class="info-table">
          <tr>
            <th>정책명</th>
            <td>{{ uppol.policy_title }}</td>
          </tr>
          <tr>
            <th>이용 기준</th>
            <td>{{ uppol.standard === "U" ? "사용자" : "에이전트" }}</td>
          </tr>
          <tr>
            <th>최대 사용 수</th>
            <td v-if="uppol.max_count === 0">
              제한 없음
            </td>
            <td v-if="uppol.max_count !== 0">{{ uppol.max_count }} 명</td>
          </tr>
          <tr>
            <th>이용 개월 수</th>
            <td>
              <input type="number" min="1" id="usemonth" v-model="usemonth" />
              개월
              <div style="font-size:13px; color:red;">
                * 개월 수는 30일로 계산됩니다.
              </div>
            </td>
          </tr>
          <tr>
            <th>갱신 시작일</th>
            <td>{{ getStartDate() }}</td>
          </tr>
          <tr>
            <th>구독 종료일</th>
            <td>{{ getEndDate() }}</td>
          </tr>
          <tr>
            <th>프로모션 적용</th>
            <td>
              <template
                v-if="this.$store.state.productStore.promotions.length !== 0"
              >
                <ul>
                  <li
                    v-for="promotion in promotions"
                    v-bind:key="promotion.no"
                    style="margin-bottom:10px"
                  >
                    프로모션 명 : {{ promotion.title }} <br />
                    할인률 : {{ promotion.discount }}% <br />
                    할인 금액 :
                    <span style="color: red"
                      >{{
                        numberWithCommas(
                          promotion.policy_price *
                            (promotion.discount / 100) *
                            usemonth
                        )
                      }}원</span
                    >
                  </li>
                </ul>
              </template>
              <template
                v-if="this.$store.state.productStore.promotions.length === 0"
              >
                적용된 프로모션이 없습니다.
              </template>
            </td>
          </tr>
        </table>

        <hr style="border-top: 1px solid black;" />
        <div
          id="price-title"
          style="width: 80%; text-align:right; font-size:17px; font-weight: normal; float:left"
        >
          <p style="color: gray">정가 :</p>
          <p style="color: red">할인 금액 :</p>
        </div>
        <div
          id="final-price"
          style="width: 20%; text-align:right; font-size:17px; font-weight: normal; float:left"
        >
          <p style="color: gray">
            {{ numberWithCommas(uppol.price * usemonth) }}원
          </p>
          <p style="color: red">- {{ numberWithCommas(alldiscount) }}원</p>
        </div>
        <br /><br /><br />
        <hr style="border-top: 1px solid gray;" />
        <div
          id="price-title"
          style="width: 80%; text-align:right; font-size:17px; font-weight: normal; float:left"
        >
          <p style="color: black">결제 금액 :</p>
        </div>
        <div
          id="final-price"
          style="width: 20%; text-align:right; font-size:17px; font-weight: normal; float:left"
        >
          {{ numberWithCommas(uppol.price * usemonth - alldiscount) }}원
        </div>
        <br /><br />

        <hr style="border-top: 1px solid #ccc;" />
        <div class="selPayM" align="center">
          <h4 style="font-weight: normal;">결제 방식 선택</h4>
          <button
            id="cards"
            @click="selPayM('cards')"
            :style="{ background: cardsCol }"
          >
            신용카드
          </button>
          <button
            id="trans"
            @click="selPayM('trans')"
            :style="{ background: transCol }"
          >
            실시간계좌이체
          </button>
        </div>
        <hr style="border-top: 1px solid #ccc;" />
        <br />
        <p align="center" style="font-size:17px">
          <input type="checkbox" v-model="chk" /> 다음 결제정보에 대해 이해하고
          있으며, 결제를 진행합니다.
        </p>
        <div align="center">
          <button style="width:110px" @click="gopay()">
            결제하기
          </button>
        </div>
        <br />
      </template>
    </div>
  </div>
</template>

<script>
import axios from "axios";
// import promotionPolicyCheckPNo from "../../../../api/shr/promotion";
var IMP = window.IMP; // 생략해도 괜찮습니다.
IMP.init("imp67730889"); // "imp00000000" 대신 발급받은 "가맹점 식별코드"를 사용합니다.

export default {
  data() {
    return {
      usemonth: 1,
      enddate: "",
      chk: false,
      paymethod: "",
      year: "", // today.getFullYear(); // 년도
      month: "", // today.getMonth() + 1;  // 월
      date: "", // today.getDate();  // 날짜

      // 결제 방식 선택 배경색
      cardsCol: "",
      transCol: "",
    };
  },
  methods: {
    getStartDate: function() {
      return (
        this.leadingZeros(this.startdate.getFullYear(), 4) +
        "-" +
        this.leadingZeros(this.startdate.getMonth() + 1, 2) +
        "-" +
        this.leadingZeros(this.startdate.getDate(), 2)
      );
    },
    getEndDate: function() {
      this.enddate.setDate(this.enddate.getDate() + this.usemonth * 30);

      return (
        this.leadingZeros(this.enddate.getFullYear(), 4) +
        "-" +
        this.leadingZeros(this.enddate.getMonth() + 1, 2) +
        "-" +
        this.leadingZeros(this.enddate.getDate(), 2)
      );
    },
    leadingZeros(n, digits) {
      var zero = "";
      n = n.toString();

      if (n.length < digits) {
        for (var i = 0; i < digits - n.length; i++) zero += "0";
      }
      return zero + n;
    },
    numberWithCommas(x) {
      return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    getDate(value) {
      var yyyy = value.getFullYear().toString();
      var mm = (value.getMonth() + 1).toString();
      var dd = value.getDate().toString();
      var hh = value.getHours().toString();
      var mn = value.getMinutes().toString();
      var ss = value.getSeconds().toString();
      return (
        yyyy +
        "-" +
        (mm[1] ? mm : "0" + mm[0]) +
        "-" +
        (dd[1] ? dd : "0" + dd[0]) +
        " " +
        (hh[1] ? hh : "0" + hh[0]) +
        ":" +
        (mn[1] ? mn : "0" + mn[0]) +
        ":" +
        (ss[1] ? ss : "0" + ss[0])
      );
    },
    gopay() {
      // 결제 수행
      console.log(this.getDate(this.enddate));
      if (this.paymethod === "") {
        alert("결제 방식을 선택해주세요.");
      } else if (this.chk === false) {
        alert("결제정보를 확인하고 동의해주세요.");
      } else if (this.chk === true && this.paymethod !== "") {
        // IMP.request_pay(param, callback) 호출
        IMP.request_pay(
          {
            // param
            pg: "inicis",
            pay_method: this.paymethod,
            merchant_uid: "INZENT_Solutions" + new Date().getTime(),
            name: this.uppol.policy_title + "/mxc" + this.uppol.max_count,
            amount: 100, // this.type == 'U' ? this.promotions[i].price * (this.promotions[i].discount/100) * this.usemonth : this.promotions[i].price * (this.promotions[i].discount/100) * this.usemonth,
            buyer_email: "iamport@siot.do",
            buyer_name: "구매자이름",
            buyer_tel: "010-1234-5678",
            buyer_addr: "서울특별시 강남구 삼성동",
            buyer_postcode: "123-456",
          },
          async (rsp) => {
            // callback
            var msg = "";
            // 결제 성공 시 로직,
            if (rsp.success) {
              // TODO
              // 1. 주문내역 추가
              await axios
                .get("/api/addOrderList", {
                  params: {
                    policyNo: this.uppol.no,
                    userNo: this.userInfo.no,
                    monthCount: this.usemonth,
                    startDate: this.startdate,
                    endDate: this.enddate,
                    orgPrice: this.uppol.price * this.usemonth,
                    dcPrice: this.alldiscount,
                    totalPrice:
                      this.uppol.price * this.usemonth - this.alldiscount,
                    monthPay: false,
                  },
                })
                .then((res) => {
                  console.log(res.data);
                });

              // 2. 라이센스 갱신
              await axios
                .get("/api/updateLicense", {
                  params: {
                    licenseNo: this.lcno,
                    enddate: this.getDate(this.enddate),
                  },
                })
                .then((res) => {
                  console.log(res.data);
                });

              // 갱신 알림 제거
              axios
                .get("/api/alertLicenseOver", {
                  params: {
                    userAdminNo: this.userInfo.no,
                  },
                })
                .then((res) => {
                  this.$store.commit("licenseStore/SELECT_AOL", res.data);
                  console.log(res.data);
                });

              alert("구독 갱신이 완료되었습니다.");
              this.$router.push("/license/list");

              // 결제 실패 시 로직,
            } else {
              msg = "결제에 실패하였습니다.\n";
              msg += "내용 : " + rsp.error_msg;
              alert(msg);
            }
          }
        );
      }
    },
    selPayM(payM) {
      this.paymethod = payM; // 결제 방식 선택
      switch (payM) {
        case "cards":
          this.cardsCol = "#99CCCC";
          this.transCol = "";
          break;
        case "trans":
          this.cardsCol = "";
          this.transCol = "#99CCCC";
          break;
      }
    },
  },
  computed: {
    ued: function() {
      return this.$store.state.licenseStore.updateEndDate;
    },
    lcno: function() {
      return this.$store.state.licenseStore.lcno;
    },
    uppol: function() {
      return this.$store.state.licenseStore.updatePolicy;
    },
    startdate: function() {
      return this.$store.state.productStore.startdate;
    },
    promotions: function() {
      return this.$store.state.productStore.promotions;
    },
    discountTotal_policy() {
      let sum = 0;
      Object.values(this.promotionPolicyList).filter((item) => {
        sum += Number(item.policy_price * (item.discount / 100));
      });
      return sum * this.usemonth;
    },
    alldiscount: function() {
      // 할인액 산정
      this.$store.commit("productStore/DEL_ALLDC");

      for (var i = 0; i < this.promotions.length; i++) {
        this.$store.commit(
          "productStore/ADD_ALLDC",
          this.promotions[i].policy_price *
            (this.promotions[i].discount / 100) *
            this.usemonth
        );
      }

      return this.$store.state.productStore.alldiscount;
    },
    userInfo: function() {
      return this.$store.state.userinfo.userInfo;
    },
  },
  async created() {
    console.log(this.ued);
    // 구독 시작일, 종료일 설정
    this.$store.commit("productStore/SET_STARTDATE", new Date(this.ued));
    this.enddate = new Date(this.ued);
    // alert((new Date).toISOString());
  },
  beforeUpdate() {
    this.enddate = new Date(this.ued);
  },
};
</script>

<style scoped>
h1 {
  font-size: 4.5rem;
  color: #000000d9;
}
.tip {
  border: 2px solid gray;
  color: red;
  font-size: 15px;
  padding: 20px;
  border-radius: 5px;
}
.info-border {
  border: 2px solid lightgray;
  border-radius: 5px;
  padding: 10px;
}
.info-table {
  border-top: 2px solid #ccc;
  border-bottom: 2px solid #ccc;
  width: 100%;
  max-width: 100%;
  border-spacing: 0;
  border-collapse: collapse;
  table-layout: fixed;
}
.info-table tr {
  border-bottom: 1px solid #ccc;
  font-size: 15px;
}
.info-table th {
  background: #eaeaea;
  font-weight: normal;
  border: none;
  width: 20%;
  text-align: center;
  padding: 15px;
}
.info-table td {
  padding: 15px;
  font-weight: normal;
}
.info-table input {
  width: 100px;
}
.selPayM button {
  width: 20%;
  height: 80px;
  margin: 0 20px 0;
  background: gray;
  font-size: 20px;
}
.nav-tabs li.active a {
  background-color: #d9edf7;
}
</style>
