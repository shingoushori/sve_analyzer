<template>
  <div class="svg-area">
    <svg
      :width="width"
      :height="height"
      version="1.1"
      xmlns="http://www.w3.org/2000/svg"
      ref="svg"
      :viewBox="viewBox"
      preserveAspectRatio="xMidYMid meet"
    >
      <g :transform="transform">
        <g
          v-if="config.line.width"
          class="line-group"
          fill="none"
          :stroke-width="config.line.width"
        >
          <DiagramLine v-for="line in lines" :key="line.id" :line="line" :config="config" />
        </g>
        <g
          v-if="config.stop.width"
          class="stop-mark-group"
          fill="white"
          :stroke="config.stop.stroke"
          :stroke-width="config.stop.width"
        >
          <DiagramStopMark
            v-for="(stop,index) in stops"
            :key="index"
            :stop="stop"
            :config="config"
            :stroke="config.stop.stroke"
            :fill="config.stop.fill"
          />
        </g>
        <g
          v-if="config.stop_label.size"
          class="station-group"
          :font-family="font_family"
          :font-size="config.stop_label.size"
          :fill="config.stop_label.fill"
        >
          <DiagramStopLabel
            v-for="(stop_label,index) in stop_labels"
            :key="index"
            :stop_label="stop_label"
            :config="config"
          />
        </g>
        <g>
          <rect
            x="0"
            y="0"
            :width="(Number(this.info.map.width) * this.config.scale) / 100"
            :height="(Number(this.info.map.depth) * this.config.scale) / 100"
            fill="none"
          />
        </g>
      </g>
    </svg>
    <div class="control-box d-flex">
      <b-form-checkbox
        v-model="adjust_scale"
        class="mr-3 mb-2"
        switch
      >表示サイズ: {{ adjust_scale ? '自動' : '等倍'}}</b-form-checkbox>
      <div>出力サイズ: {{ render_size }}</div>
    </div>
  </div>
</template>
<script>
import diagramService from "../../../services/diagram";
import relationService from "../../../services/relation";

export default {
  props: ["info", "config", "selected_lines"],
  data() {
    return {
      adjust_scale: true
    };
  },
  computed: {
    width() {
      return this.adjust_scale ? "100vw" : this.view_width;
    },
    height() {
      return this.adjust_scale ? "90vh" : this.view_height;
    },
    view_width() {
      return (
        (Number(this.info.map.width) * this.config.scale) / 100 +
        this.config.padding * 2
      );
    },
    view_height() {
      return (
        (Number(this.info.map.depth) * this.config.scale) / 100 +
        this.config.padding * 2
      );
    },
    viewBox() {
      return `0 0 ${this.view_width} ${this.view_height}`;
    },
    render_size() {
      return `${this.view_width} x ${this.view_height} px`;
    },
    transform() {
      return `translate(${this.config.padding}, ${this.config.padding})`;
    },
    lines() {
      return this.info.lines
        .filter(l => this.selected_lines.includes(l.id))
        .map(l => diagramService.toRenderLine(l))
        .map(l => {
          l.color = relationService.getPlayerColor1ByPlayerId(
            this.info.players,
            l.player_id
          );
          return l;
        });
    },
    stops() {
      return this.lines.map(l => l.stops).flat();
    },
    stop_labels() {
      const stop_labels = [];
      this.stops.map(stop => {
        if (
          stop_labels.findIndex(sl => sl.station_id === stop.station_id) === -1
        ) {
          stop_labels.push({
            station_id: stop.station_id,
            coordinate: stop.coordinate,
            name: relationService.getStationNameById(
              this.info.stations,
              stop.station_id
            )
          });
        }
      });
      return stop_labels;
    },
    font_family() {
      return "'Hiragino Kaku Gothic Pro','ヒラギノ角ゴ Pro W3','メイリオ',Meiryo,'ＭＳ Ｐゴシック',sans-serif'";
    }
  }
};
</script>
<style lang="scss" scoped>
.control-box {
  position: fixed;
  left: 1rem;
  bottom: 1.5rem;
  padding: 0.2rem;
  background-color: #fffc;
}
</style>