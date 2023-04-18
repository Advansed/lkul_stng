<template lang="pug">
.agreements-address-counter
  .history__table(v-if="!isMobile")
    .history__table-row.history__table-row_heading
      .history__table-cell № счетчика
      .history__table-cell Показания счетчика
      .history__table-cell Дата поверки
      .history__table-cell Новые показания
    .history__table-row(v-for="item in records", :key="item.id")
      .history__table-cell {{ item.counterId }}
      .history__table-cell {{ item.old_indication }}
      .history__table-cell {{ item.date }}
        //div(:style="{color:getColor(item.date)}") {{ item.date }}
      .history__table-cell
        input(
          type="number",
          v-model="item.value",
          placeholder="Введите показания счетчика"
        )
  .mobile-table.mobile-table_variant(v-if="isMobile")
    .mobile-table__item(v-for="item in records", :key="item.id")
      .mobile-table__title {{ item.counterId }}
      .mobile-table__field
        .mobile-table__key Показания счетчика
        .mobile-table__value {{ item.old_indication }}
      .mobile-table__field
        .mobile-table__key Дата поверки
        .mobile-table__value {{ item.date }}
          //div(:style="{color:getColor(item.date)}") {{ item.date }}
      .mobile-table__field
        .mobile-table__key Новые показания
        .mobile-table__value
          input(
            type="number",
            v-model="item.value",
            placeholder="Введите показания"
          )
  .notice(v-for="item in notice", :key="item.id")
    .notice_p(v-if="!item.error")  - {{ item.message }}
    .notice_red(v-if="item.error") - {{ item.message }}
  .bills-counter__footer.bills-counter__footer_variant
    .bills-counter__description В случае неправильного ввода показаний, следует обратиться в отдел по работе с предприятиями УГРС по телефонам: 46-00-62, 46-00-73, 46-00-20, 46-00-25
    .bills-counter__submit
      ButtonClick(size="small", @click="sendIndication") Подтвердить
</template>

<script>
import { Back, ButtonClick } from "@/components";
import DatePicker from "vue2-datepicker";
import moment from "moment";
import Vue from "vue";

export default {
  name: "AgreementsAddressesCounters",
  data() {
    return {
      records: [],
      notice: "",
      objectId: "",
    };
  },
  components: {
    Back,
    ButtonClick,
    DatePicker,
  },
  methods: {
    formatDate(date) {
      return moment(date).format("DD.MM.YYYY");
    },
    getColor(date) {
      let deadline = moment(date).subtract(7, "days");
      return deadline >= moment() ? "#7F8DA8" : "#C10909";
    },
    sendIndication() {

        this.$store.dispatch("sendIndication", this.records).then((res) => {
          this.notice = res;
        });

    },
  },
  computed: {
    isMobile() {
      return screen.width < 760;
    },
    getCounters() {
      return this.$store.getters.getCurrentAgreement;
    },
    getBills() {
      return this.getCounters.objects.filter(
        (el) => el.objectId || this.$data.objectId === this.$data.objectId
      );
    },
    getNotice() {
      return this.$store.getters.getNotice;
    }
  },
  mounted() {
    let counters = this.getCounters.objects.filter(
      (el) =>
        el.objectId ===
        this.$route.params
          .address 
    );
    if (counters.length == 0) {
      counters = this.getCounters.objects.filter(
        (el) => el.objectId === Vue.cookie.get("objectId")
      );
    }
    console.log(counters, "test");
    this.$data.objectId = Vue.cookie.get("objectId");
    let test = false;
    let end = new Date((new Date).setMonth((new Date).getMonth() + 3 ))
    console.log(end)
    if (counters.length !== 0) {
      test = true;
      if (test === true) {
        Vue.cookie.set("objectId", this.$route.params.address, {
          expires: "2h",
        });
      }
      let i = 0;
      for (let counter of counters[0].counters) { i = i++;
        this.records.push({
          counterId: counter.counter,
          counterNumber: counter.Id,
          value: "",
          old_indication: counter.Indication,
          date: counter.dateEnd,
          name: counter["counter"],
          agreementId: this.$route.params.address,
          objectId: counter.objectId,
        });

        if(counter.date < end){
          this.notice = [...this.notice, {id: i, error: false, message: "Счетчик " + counter.counter + " - "
                  + "Срок поверки счетчика истекает, необходимо произвести поверку или замену счетчика до " + counter.date?.toISOString().split('T')[0]
									+ " с предварительным снятием пломбы в присутствии уполномоченного представителя УСД, сдать счетчик на приемку к " 
									+ "коммерческому учету уполномоченному представителю УСД после поверки или замены"}]
        }
      }
    }
    console.log(this.notice)
    if(this.getNotice != "")
      this.notice = this.getNotice
      console.log(this.notice)
  },
  
};
</script>

<style lang="sass">
@import ~@/assets/styles/vars

.agreements-address-counter
.notice
  padding: 24px 24px 0
  color: #3F64A9
  .notice_red
    margin: 0
    margin-left: 2%
    font-size: 14px
    color: #ff0000
  .notice_p
    margin: 0
    margin-left: 2%
    font-size: 14px
</style>
