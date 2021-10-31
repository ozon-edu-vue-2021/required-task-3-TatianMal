<template>
  <div class="map">
    <h3>Карта офиса</h3>

    <div v-if="!isLoading" class="map-root">
      <MapSVG ref="map" />
      <TableSVG v-show="false" ref="table" />
    </div>
    <div v-else>Loading...</div>
  </div>
</template>

<script>
import { select } from "d3";

import MapSVG from "@/assets/images/map.svg";
import TableSVG from "@/assets/images/workPlace.svg";

import legend from "@/assets/data/legend.json";
import tables from "@/assets/data/tables.json";

export default {
  components: {
    MapSVG,
    TableSVG,
  },
  data() {
    return {
      isLoading: false,
      map: null,
      groupMap: null,
      tableTemplate: null,
      tables: [],
    };
  },
  created() {
    this.tables = tables;
  },
  mounted() {
    this.isLoading = true;

    this.map = select(this.$refs.map);
    this.groupMap = this.map.select("g");
    this.tableTemplate = select(this.$refs.table);

    if (this.groupMap) {
      this.drawTables();
    } else {
      alert("SVG is incorrect");
      console.error("map", this.map);
    }

    this.isLoading = false;
  },
  methods: {
    drawTables() {
      const svgTablesGroupPlace = this.groupMap
        .append("g")
        .classed("groupPlaces", true);

      this.tables.forEach((table) => {
        const targetSeat = svgTablesGroupPlace
          .append("g")
          .attr("transform", `translate(${table.x}, ${table.y}) scale(0.5)`)
          .attr("id", table._id)
          .classed("employer-place", true);

        targetSeat
          .append("g")
          .attr("transform", `rotate(${table.rotate || 0})`)
          .attr("group_id", table.group_id)
          .classed("table", true)
          .html(this.tableTemplate.html())
          .attr(
            "fill",
            legend.find((it) => it.group_id === table.group_id)?.color ??
              "transparent"
          )
          .on("click", () => {
            this.selectTable(table._id);
          });
      });
    },
    selectTable(tableId) {
      console.log(tableId);
      this.$emit("selectTable", tableId);
    },
    unselectTable() {
      this.$emit("unselectTable");
    },
  },
};
</script>

<style scoped>
.map {
  height: 100%;
  width: 100%;
  padding: 24px;
  overflow: hidden;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

.map-root {
  height: 100%;
  width: 100%;
  overflow: hidden;
  box-sizing: border-box;
}

h3 {
  margin-top: 0px;
}

::v-deep svg {
  height: 100%;
  width: 100%;
}

::v-deep .table {
  cursor: pointer;
}
</style>
