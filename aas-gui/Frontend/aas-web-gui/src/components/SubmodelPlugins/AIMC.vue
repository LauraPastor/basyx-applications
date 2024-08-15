<template>
  <v-container fluid class="pa-0">
    <!-- Header -->
    <v-card class="mb-4">
      <v-card-title>
        <div class="text-subtitle-1">
          {{ "Asset Interfaces Mapping Configuration:" }}
        </div>
      </v-card-title>
    </v-card>
    <v-expansion-panels>
      <!-- Iterate through each MappingSourceSinkRelation -->
      <v-expansion-panel
        v-for="(relation, index) in mappingSourceSinkRelations"
        :key="index"
      >
        <v-expansion-panel-title v-if="relation.first.length > 4">
          <div class="text-subtitle-1">{{ relation.first[4].value }}</div>
        </v-expansion-panel-title>
        <v-expansion-panel-text>
          <!-- Display first and second endpoints -->
          <v-card class="pb-4">
            <v-row flex align="center" justify="center">
              <v-col cols="12" md="11">
                <!-- First Endpoint -->
                <div class="text-subtitle-1">Asset Interfaces Description</div>
                <v-list dense>
                  <v-list-item-group>
                    <v-list-item
                      v-for="(key, keyIndex) in relation.first"
                      :key="keyIndex"
                    >
                      <v-list-item-content>
                        <v-list-item-title>{{ key.type }}</v-list-item-title>
                        <v-list-item-subtitle>{{
                          key.value
                        }}</v-list-item-subtitle>
                      </v-list-item-content>
                    </v-list-item>
                  </v-list-item-group>
                </v-list>

                <!-- Second Endpoint -->
                <div class="text-subtitle-1 mt-4">Sensor Data</div>
                <v-list dense>
                  <v-list-item-group>
                    <v-list-item
                      v-for="(key, keyIndex) in relation.second"
                      :key="keyIndex"
                    >
                      <v-list-item-content>
                        <v-list-item-title>{{ key.type }}</v-list-item-title>
                        <v-list-item-subtitle>{{
                          key.value
                        }}</v-list-item-subtitle>
                      </v-list-item-content>
                    </v-list-item>
                  </v-list-item-group>
                </v-list>
              </v-col>
            </v-row>
          </v-card>
        </v-expansion-panel-text>
      </v-expansion-panel>
    </v-expansion-panels>
    <!-- Handle case when no relations are available -->
    <v-card v-if="!mappingSourceSinkRelations.length" class="mb-4 py-8">
      <v-row flex align="center" justify="center">
        <div>No Mapping Source Sink Relations available</div>
      </v-row>
    </v-card>
  </v-container>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { useTheme } from "vuetify";
import { useNavigationStore } from "@/store/NavigationStore";
import { useAASStore } from "@/store/AASDataStore";
import RequestHandling from "../../mixins/RequestHandling";
import SubmodelElementHandling from "../../mixins/SubmodelElementHandling";

interface ModelReference {
  type: string;
  value: string;
}

interface MappingSourceSinkRelation {
  first: ModelReference[];
  second: ModelReference[];
}

export default defineComponent({
  name: "AssetInterfacesMappingConfiguration",
  components: {
    RequestHandling, // Mixin to handle the requests to the AAS
  },
  mixins: [RequestHandling, SubmodelElementHandling],
  props: {
    submodelElementData: {
      type: Object,
      required: true,
    },
  },
  setup() {
    const theme = useTheme();
    const navigationStore = useNavigationStore();
    const aasStore = useAASStore();

    return {
      theme, // Theme Object
      navigationStore, // NavigationStore Object
      aasStore, // AASStore Object
    };
  },
  data() {
    return {
      mappingSourceSinkRelations: [] as Array<MappingSourceSinkRelation>,
    };
  },
  mounted() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      if (!this.submodelElementData?.submodelElements) {
        console.warn("SubmodelElementData or submodelElements is not defined");
        return;
      }
      console.log(
        "Component Mounted. SubmodelElementData:",
        this.submodelElementData
      );
      // Find MappingSourceSinkRelations
      const mappingConfigurations =
        this.submodelElementData.submodelElements.find(
          (element: any) => element.idShort === "MappingConfigurations"
        );

      if (mappingConfigurations) {
        mappingConfigurations.value.forEach((config: any) => {
          const sourceSinkRelations = config.value.find(
            (element: any) => element.idShort === "MappingSourceSinkRelations"
          );

          if (sourceSinkRelations) {
            const relations = sourceSinkRelations.value.map((relation: any) => {
              return {
                first: relation.first.keys,
                second: relation.second.keys,
              } as MappingSourceSinkRelation;
            });

            this.mappingSourceSinkRelations.push(...relations);
          }
        });
      }
    },
  },
});
</script>
