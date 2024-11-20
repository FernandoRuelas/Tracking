<template>
  <v-row>
    <v-col cols="12" sm="12" md="5">
      <v-card elevation="2">
        <v-card-text>
          <div id="map"></div>
        </v-card-text>
      </v-card>
    </v-col>
    <v-col cols="12" sm="12" md="7">
      <v-card elevation="2">
        <v-card-text>
          <v-row justify="space-between" class="mb-1">
            <v-col cols="6">
              <v-text-field
                v-model="table.search"
                append-icon="mdi-magnify"
                label="Search"
                solo
                flat
                outline
                background-color="#e7eaf3"
                hide-details
                dense
              ></v-text-field>
            </v-col>

            <v-col cols="6" class="d-flex justify-end" style="gap: 10px">
              <v-btn color="primary" dense @click="add" id="btnAdd">
                <i class="fas fa-plus text-white mr-2"></i>
                <span>Add</span>
              </v-btn>
            </v-col>
          </v-row>
          <v-data-table
            :headers="table.headers"
            :items="table.items"
            :search="table.search"
            class="header-tabla addin-table-fixed"
          >
            <template v-slot:item.referens="{ item }">
              {{ item.referens || "-" }}
            </template>

            <template v-slot:item.actions="{ item }">
              <div id="options">
                <v-tooltip bottom>
                  <template v-slot:activator="{ on, attrs }">
                    <v-btn
                      icon
                      color="primary"
                      v-bind="attrs"
                      v-on="on"
                      @click.stop="zoomToZone(item, 'map')"
                    >
                      <v-icon small>fas fa-location</v-icon>
                    </v-btn>
                  </template>
                  <span>Location</span>
                </v-tooltip>
                <v-tooltip bottom>
                  <template v-slot:activator="{ on }">
                    <v-btn
                      icon
                      v-on="on"
                      class="mr-2"
                      color="primary"
                      @click="edit(item)"
                    >
                      <i class="fas fa-pencil"></i>
                    </v-btn>
                  </template>
                  <span>Edit</span>
                </v-tooltip>
                <v-tooltip bottom>
                  <template v-slot:activator="{ on }">
                    <v-btn
                      icon
                      v-on="on"
                      class="mr-2"
                      color="primary"
                      @click="deleteElement(item.id)"
                    >
                      <i class="fas fa-trash"></i>
                    </v-btn>
                  </template>
                  <span>Delete</span>
                </v-tooltip>
              </div>
            </template>
          </v-data-table>
        </v-card-text>
      </v-card>
    </v-col>
    <dialog-add
      :dialog="isOpenDialog"
      @update:dialog="isOpenDialog = $event"
      :isEdit="isEdit"
      :item="elementToEdit"
      @save="saveElement"
    >
    </dialog-add>
  </v-row>
</template>

<style>
.header-tabla th {
  background: #25477b !important;
  color: white !important;
}

#map {
  height: 81vh;
  width: 100%;
}

/*********************Tooltip del markador **************/
.gm-style-iw {
  color: white;
  max-width: 270px !important;
  background-color: #145fb4 !important;
  padding: 0 10px 10px !important;
}

.gm-style-iw-d {
  overflow: hidden !important;
}

.gm-style .gm-style-iw-t::after {
  box-shadow: none !important;
  background: linear-gradient(
    45deg,
    rgb(20 95 180) 50%,
    rgb(20 95 180) 51%,
    rgb(20 95 180) 100%
  ) !important;
}

.gm-style .gm-style-iw-tc::after {
  background-color: #145fb4 !important;
}

.gm-ui-hover-effect > span {
  background-color: white !important;
  margin: 0 !important;
}

.gm-style-iw-chr button {
  width: 20px !important;
  height: 25px !important;
}
</style>

<script>
/* global google */
import marcador from "../assets/marcadorVerde.svg";
import dialogAdd from "@/components/ModalAdd.vue";
export default {
  name: "HomeView",
  components: {
    dialogAdd,
  },
  data: () => ({
    table: {
      items: [
        {
          id: 1,
          name: "Fernando",
          direction: "Bajos pirineos 13",
          referens: "Enfrente de abarrotes",
          sales: 20,
          lat: 29.072967,
          lng: -110.955919,
        },
        {
          id: 2,
          name: "Juan",
          direction: "Plaza Zaragoza",
          referens: null,
          sales: 10,
          lat: 29.072967,
          lng: -110.955919,
        },
        {
          id: 3,
          name: "Eduardo",
          direction: "Estadio Sonora",
          referens: null,
          sales: 50,
          lat: 29.080856,
          lng: -110.967843,
        },
        {
          id: 4,
          name: "Matias",
          direction: "Parque Madero",
          referens: null,
          sales: 35,
          lat: 29.072297,
          lng: -110.955159,
        },
        {
          id: 5,
          name: "Favian",
          direction: "Centro de Gobierno",
          referens: null,
          sales: 12,
          lat: 29.085833,
          lng: -110.961389,
        },
      ],
      headers: [
        { text: "Name", value: "name" },
        { text: "Direction", value: "direction" },
        { text: "Referens", value: "referens" },
        { text: "Sales", value: "sales" },
        { text: "Actions", value: "actions", sortable: false, align: "center" },
      ],
      search: null,
    },
    maps: {
      map: null,
      dialog: null,
    },
    markers: {
      map: [],
      dialog: [],
    },
    infowindowMarkers: {
      map: [],
      dialog: [],
    },
    elementToEdit: {},
    isOpenDialog: false,
    isEdit: false,
    obj: {
      lat: null,
      lng: null,
    },

    clousters: {
      map: null,
      dialog: null,
    },
  }),
  mounted() {
    window.initMap = () => this.initMap(null, null, "map");
    window.app = this;
    this.tour();
  },
  methods: {
    clearTourAndRefresh() {
      // Elimina el item 'tour' del localStorage
      localStorage.removeItem("tour");

      // Actualiza la página
      window.location.reload();
    },
    
    tour() {
      if (localStorage.getItem("tour")) {
        return;
      }

      const paso1 = "Add a New Point of Sale!";
      const paso1Des = "Click this button to easily add a new point of sale.";

      const paso2 = "Complete the Information";
      const paso2Des =
        "Fill out all the required fields and save the information to view it on your map and main table.";

      const paso3 = "Available Options";
      const paso3Des =
        "Here you'll find tools to edit, delete, and search for points of sale based on your needs.";

      const paso4 = "Remember";
      const paso4Des =
        "You can revisit the tour whenever you need by clicking this button.";

      const fin = "Thank you for completing the tour! We hope it was helpful.";

      const inicio = "Welcome to the Point of Sale Management Demo.";
      const inicioDes =
        "Would you like me to briefly show you the project's features? It'll be quick, and you can cancel the tour at any time.";

      const driver = window.driver.js.driver;

      const steps = [
        {
          element: null,
          popover: {
            title: inicio,
            description: inicioDes,
            side: "center",
            align: "center",
            disableButtons: [],
          },
        },

        {
          element: "#btnAdd",
          popover: {
            title: paso1,
            description: paso1Des,
            side: "left",
            align: "start",
            onNextClick: () => {
              document.getElementById("btnAdd").click();
              setTimeout(() => {
                driverObj.moveNext();
              }, 400);
            },
          },
        },
        {
          element: "#modal",
          popover: {
            title: paso2,
            description: paso2Des,
            side: "left",
            align: "start",
            onNextClick: () => {
              document.getElementById("closeModal").click();
              setTimeout(() => {
                driverObj.moveNext();
              }, 400);
            },
          },
        },

        {
          element: "#options",
          popover: {
            title: paso3,
            description: paso3Des,
            side: "left",
            align: "start",
          },
        },

        {
          element: "#recordar",
          popover: {
            title: paso4,
            description: paso4Des,
            side: "left",
            align: "start",
          },
        },

        {
          element: null,
          popover: {
            title: fin,
            side: "center",
            align: "center",
          },
        },
      ];

      const driverObj = driver({
        steps: steps,
        showPrevButton: true,
        showProgress: true,
        popoverClass: "my-custom-popover-class",
        onDestroyStarted: () => {
          localStorage.setItem("tour", true);

          if (
            !driverObj.hasNextStep() ||
            confirm("Do you want finish the tour?")
          ) {
            driverObj.destroy();
          }
        },
      });

      driverObj.drive();
    },

    async initMap(lat, lng, idContainer) {
      if (!window.google || !window.google.maps) {
        console.error("Google Maps API no está disponible");
        return;
      }

      const position = { lat: lat || 24.8591662, lng: lng || -101.9134626 };
      const zoom = lat === 24.8591662 ? 4.79 : 11.75;

      this.maps[idContainer] = new google.maps.Map(
        document.getElementById(idContainer),
        {
          zoom: zoom,
          center: position,
          mapTypeId: "roadmap",
          fullscreenControl: false,
          disableDefaultUI: false,
          zoomControl: true,
          mapTypeControl: false,
          scaleControl: false,
          rotateControl: false,
        }
      );

      idContainer == "dialog" &&
        this.maps["dialog"].addListener("click", (event) => {
          this.onMapClick(event);
        });
      if (!lat) {
        this.solicitarPermisoUbicacion(idContainer);
      }

      idContainer == "map" && this.addMarkersToMap();
    },

    addMarkersToMap() {
      this.table.items.forEach((user) => {
        this.addMarker(user, "map");
      });
    },

    addMarker(user, map) {
      if (user.lat && user.lng) {
        const position = {
          lat: parseFloat(user.lat),
          lng: parseFloat(user.lng),
        };

        const marker = new google.maps.Marker({
          id: user.id,
          position: position,
          map: this.maps[map],
          icon: {
            url: marcador,
            fillColor: "#00205c",
            scaledSize: new google.maps.Size(24, 24),
          },
        });

        if (user.name) {
          // Crear un tooltip (infowindow)
          const infowindow = new google.maps.InfoWindow({
            content: `<div><strong>${user.name}</strong><br>${user.direction}</div>`,
          });

          // Mostrar el tooltip al hacer clic en el marcador
          marker.addListener("click", () => {
            infowindow.open(this.maps[map], marker);
          });
          infowindow.open(this.maps[map], marker);

          this.infowindowMarkers[map].push(infowindow);
        }

        this.markers[map].push(marker);
      }
    },

    deleteMarker(idMap) {
      for (let i = 0; i < this.markers[idMap].length; i++) {
        this.markers[idMap][i].setMap(null);
      }

      this.markers[idMap] = [];
    },

    deteteMarkerId(idMap, markerId) {
      if (!this.markers[idMap]) return;

      const markerIndex = this.markers[idMap].findIndex(
        (marker) => marker.id === markerId
      );

      if (markerIndex !== -1) {
        this.markers[idMap][markerIndex].setMap(null);

        this.markers[idMap].splice(markerIndex, 1);
      }
    },

    onMapClick(e) {
      // Borra el marcador anterior
      console.log("e", e);
      if (this.markers["dialog"].length > 0) {
        this.deleteMarker("dialog");
      }

      // Guarda la latitud y longitud
      this.obj.lat = e.latLng.lat();
      this.obj.lng = e.latLng.lng();

      // Crea un nuevo marcador
      this.addMarker(this.obj, "dialog");
    },

    solicitarPermisoUbicacion(idContainer) {
      const self = this;
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          function (position) {
            self.initMap(
              position.coords.latitude,
              position.coords.longitude,
              idContainer
            );
          },
          function (error) {
            console.error("Location request denied", error);
            self.initMap(24.8591662, -101.9134626, idContainer);
          }
        );
      } else {
        self.initMap(24.8591662, -101.9134626, idContainer);
      }
    },

    zoomToZone(item, map) {
      this.maps[map].setZoom(16.5); // Ajusta el zoom según sea necesario
      this.maps[map].setCenter({
        lat: item.lat,
        lng: item.lng,
      });
      this.openTooltipsZones(item, map);
    },

    openTooltipsZones(item, map) {
      const infowindow = this.infowindowMarkers[map][item.id - 1];

      if (infowindow) {
        infowindow.open(this.maps[map], this.markers[map][item.id - 1]);
      } else {
        console.error(
          `No se encontró el infowindow para el marcador con ID ${item.id}`
        );
      }
    },

    edit(item) {
      this.elementToEdit = item;
      this.isEdit = true;
      this.isOpenDialog = true;
      this.$nextTick(async () => {
        await this.initMap(null, null, "dialog");
        setTimeout(() => {
          this.addMarker(item, "dialog");
        }, 200);
      });
    },

    add() {
      this.isEdit = false;
      this.isOpenDialog = true;

      this.$nextTick(async () => {
        await this.initMap(null, null, "dialog");
      });
    },

    async saveElement(element) {
      await this.deleteMarker("dialog");
      const newItem = JSON.parse(JSON.stringify(element));
      this.isEdit ? this.editElement(newItem) : this.addElement(newItem);
      this.isOpenDialog = false;
    },

    async addElement(item) {
      item.id = this.table.items.length + 1;
      console.log("item", item);

      item.lat = this.obj.lat;
      item.lng = this.obj.lng;
      this.table.items.push(item);
      await this.deleteMarker("map");
      this.addMarkersToMap();
      this.obj.lat = null;
      this.obj.lng = null;
    },

    async editElement(element) {
      const index = this.table.items.findIndex(
        (item) => item.id === element.id
      );

      if (index !== -1) {
        const newItem = {
          ...this.table.items[index],
          ...element,
        };

        newItem.lat = this.obj.lat || newItem.lat;
        newItem.lng = this.obj.lng || newItem.lng;

        this.$set(this.table.items, index, newItem);
        if (this.obj.lat || this.obj.lng) {
          await this.deleteMarker("map");
          this.addMarkersToMap();
        }
      }
      this.elementToEdit = {};
    },

    deleteElement(id) {
      this.table.items = this.table.items.filter((item) => item.id != id);
      this.deteteMarkerId("map", id);
    },
  },
};
</script>
