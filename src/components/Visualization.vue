<template>
    <div>
        <div class="header">
            <h1>Hello, World in data</h1>
        </div>
        <b-row>
            <b-col class="slider">
                <Timeline :years="years" @change="selectedYear = $event"></Timeline>
            </b-col>
        </b-row>

        <b-row>
            <b-col cols="2" class="key">
                <Key :regions="regions" @change="selectedRegions = $event" :regionColorScale="regionColorScale"></Key>
            </b-col>
            <b-col cols="7">
                <Diagram :data="selectedYearData" :selected-regions="selectedRegions"
                         @areaSelected="selectedArea = $event"
                         :diagramDomain="diagramDomain" :regionColorScale="regionColorScale"></Diagram>
            </b-col>
            <b-col cols="3">
                <CountryDetail :data="selectedAreaYearDataSet"></CountryDetail>
            </b-col>
        </b-row>

        <b-row>
            <b-col cols="7" offset="2">
                <CountryTimeChart :data="selectedAreaData" :years="years" :selected-year="selectedYear"
                                  :diagramDomain="diagramDomain"></CountryTimeChart>
            </b-col>
        </b-row>
        <div class="copy">
            &copy; 2019 Belegarbeit PBO von Oliver von Seydlitz, Leo Lindhorst, Duc Hung Nguyen, Denis Keiling
        </div>
    </div>
</template>

<script lang="ts">
  import CountryDetail from '@/components/CountryDetail.vue';
  import CountryTimeChart from '@/components/CountryTimeChart.vue';
  import Diagram from '@/components/Diagram.vue';
  import Key from '@/components/Key.vue';
  import Timeline from '@/components/Timeline.vue';
  import { DataGroup } from '@/script/DataGroup';
  import { ItemCodes } from '@/script/DataPoint';
  import { DataProvider } from '@/script/DataProvider';
  import { DiagramDomain } from '@/script/DiagramDomain';
  import * as d3 from 'd3';
  import Vue from 'vue';
  import Component from 'vue-class-component';


  @Component({
    components: {
      CountryTimeChart,
      CountryDetail,
      Diagram,
      Key,
      Timeline,
    },
  })
  export default class Visualization extends Vue {
    public dataProvider: DataProvider | null = null;
    public selectedYear: number | null = null;
    public selectedRegions: string[] = [];
    private selectedArea: string | null = 'Germany';

    public async created() {
      this.dataProvider = await DataProvider.loadJSON();
    }

    get years(): number[] | null {
      if (!this.dataProvider) {
        return null;
      }

      return [...new Set(
        this.dataProvider.preparedData.filter(d =>
          d.values.some(v => v['Item Code'] === ItemCodes.ANEMIA_CODE)
          && d.values.some(v => v['Item Code'] === ItemCodes.PROTEIN_CODE)
          && d.values.some(v => v['Item Code'] === ItemCodes.GDP_CODE),
        )
          .map(d => d.year),
      )];
    }

    get regions(): string[] | null {
      if (!this.dataProvider) {
        return null;
      }

      return [...new Set(this.dataProvider.data.map(d => d.Region))];
    }

    get selectedYearData(): DataGroup[] | null {
      if (!this.dataProvider) {
        return null;
      }

      return this.dataProvider.preparedData.filter(({ year }) => this.selectedYear === year)
        .filter(d =>
          d.values.some(v => v['Item Code'] === ItemCodes.ANEMIA_CODE)
          && d.values.some(v => v['Item Code'] === ItemCodes.PROTEIN_CODE)
          && d.values.some(v => v['Item Code'] === ItemCodes.GDP_CODE),
        );
    }

    get selectedAreaData(): DataGroup[] | null {
      if (!this.dataProvider) {
        return null;
      }

      return this.dataProvider.preparedData.filter(({ area }) => this.selectedArea === area)
        .filter(d =>
          d.values.some(v => v['Item Code'] === ItemCodes.ANEMIA_CODE)
          && d.values.some(v => v['Item Code'] === ItemCodes.PROTEIN_CODE)
          && d.values.some(v => v['Item Code'] === ItemCodes.GDP_CODE),
        );
    }

    get selectedAreaYearDataSet(): DataGroup | null {
      if (!this.dataProvider) {
        return null;
      }

      return this.dataProvider.preparedData
        .find(({ year, area }) => this.selectedYear === year && this.selectedArea === area) || null;
    }

    get regionColorScale(): d3.ScaleOrdinal<string, string> {
      return d3.scaleOrdinal(d3.schemePaired);
    }

    get diagramDomain(): DiagramDomain | null {
      if (!this.dataProvider) {
        return null;
      }

      return {
        minimumAnemia: Number.parseFloat(this.dataProvider.getMinValue(ItemCodes.ANEMIA_CODE)!.Value),
        maximumAnemia: Number.parseFloat(this.dataProvider.getMaxValue(ItemCodes.ANEMIA_CODE)!.Value),
        minimumProtein: Number.parseFloat(this.dataProvider.getMinValue(ItemCodes.PROTEIN_CODE)!.Value),
        maximumProtein: Number.parseFloat(this.dataProvider.getMaxValue(ItemCodes.PROTEIN_CODE)!.Value),
        minimumGDP: Number.parseFloat(this.dataProvider.getMinValue(ItemCodes.GDP_CODE)!.Value),
        maximumGDP: Number.parseFloat(this.dataProvider.getMaxValue(ItemCodes.GDP_CODE)!.Value),
      };
    }
  }
</script>

<style scoped lang="scss">
    .key {
        padding-top: 50px;
        padding-left: 50px;
        display: block;
        margin-left: auto;
        margin-right: auto;

    }

    .header {
        text-align: center;
    }

    .slider {
        text-align: center;
        width: 100%;
    }

    .copy {
        text-align: center;
    }
</style>
