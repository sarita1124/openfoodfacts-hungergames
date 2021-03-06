<template>
  <div class="ui container">
    <div>
      <h2>{{$t("logos.search")}}</h2>
      <div class="ui divider hidden" />
      <form class="ui form" v-on:submit.prevent="search">
        <div class="three fields">
          <div class="field">
            <input type="text" name="value" :placeholder="$t('logos.value')" v-model.trim="value" />
          </div>
          <div class="field">
            <input type="text" name="barcode" :placeholder="$t('logos.barcode')" v-model.trim="barcode" />
          </div>
          <div class="field">
            <select class="ui fluid dropdown" v-model="type">
              <option value>{{$t("logos.type")}}</option>
              <option value="label">{{$t("logos.label")}}</option>
              <option value="brand">{{$t("logos.brand")}}</option>
              <option value="packager_code">{{$t("logos.packager_code")}}</option>
              <option value="packaging">{{$t("logos.packaging")}}</option>
              <option value="qr_code">{{$t("logos.qr_code")}}</option>
              <option value="category">{{$t("logos.category")}}</option>
              <option value="nutrition_label">{{$t("logos.nutrition_label")}}</option>
              <option value="store">{{$t("logos.store")}}</option>
            </select>
          </div>
          <div class="field">
            <input type="number" name="count" v-model.trim="count" />
          </div>
          <div class="field">
            <button type="submit" class="ui button primary">{{$t("logos.search")}}</button>
          </div>
        </div>
      </form>
      <div class="ui divider hidden" />
      <LoadingSpinner :show="loading" />
      <div v-if="!loading && logos.length > 0">
        <p>{{$t("logos.number_of_results")}} {{ resultCount }}</p>
        <LogoCardGrid :logos="logos" :selectable="false" />
      </div>
      <div v-if="!loading && logos.length == 0">
        <p>
          <strong>{{$t("logos.no_found")}}</strong>
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import robotoffService from "../robotoff";
import offService from "../off";
import LogoCardGrid from "../components/LogoCardGrid";
import LoadingSpinner from "../components/LoadingSpinner";


const transformLogo = logo => {
  logo.image.url = robotoffService.getCroppedImageUrl(
    offService.getImageUrl(logo.image.source_image),
    logo.bounding_box
  )
  return logo;
};

export default {
  name: "InsightListView",
  components: { LogoCardGrid , LoadingSpinner},
  data: function() {
    return {
      logos: [],
      value: "",
      barcode: "",
      type: "",
      resultCount: 0,
      count: 25,
      loading: false
    };
  },
  computed: {},
  methods: {
    search: function() {
      this.loading = true
      robotoffService
        .searchLogos(this.barcode, this.value, this.type, this.count)
        .then(({ data }) => {
          this.logos = data.logos.map(transformLogo);
          this.resultCount = data.count;
          this.loading = false;
        })
        .catch(() => {
          this.loading = false;
        })
    }
  },
  mounted() {
    this.search();
  }
};
</script>

<style scoped>
.borderless-button {
  color: #2185d0;
  cursor: pointer;
}

.borderless-button:hover {
  text-decoration: underline;
}
</style>
