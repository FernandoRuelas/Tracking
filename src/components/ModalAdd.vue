<template>
  <v-dialog  v-model="isOpenModal" persistent max-width="675px" attach>
    <v-card id="modal">
      <v-card-title>
        {{ isEdit ? "Edit" : "Add" }}
      </v-card-title>

      <v-card-text>
        <v-container>
          <v-form v-model="formAddElement" ref="formAddElement">
            <v-row>
              <v-col cols="12">
                <v-row class="contenedorLabelInput">
                  <p class="mb-0">Name</p>
                  <v-text-field
                    solo
                    clearable
                    required
                    v-model="obj.name"
                    background-color="#e7eaf3"
                  ></v-text-field>
                </v-row>
              </v-col>

              <v-col cols="12">
                <v-row class="contenedorLabelInput">
                  <p class="mb-0">Direction</p>
                  <v-text-field
                    solo
                    required
                    clearable
                    v-model="obj.direction"
                    background-color="#e7eaf3"
                  ></v-text-field>
                </v-row>
              </v-col>

              <v-col cols="12">
                <v-row class="contenedorLabelInput">
                  <p class="mb-0">Referens</p>
                  <v-text-field
                    solo
                    clearable
                    required
                    v-model="obj.referens"
                    background-color="#e7eaf3"
                  ></v-text-field>
                </v-row>
              </v-col>

              <v-col cols="12">
                <v-row class="contenedorLabelInput">
                  <p class="mb-0">Sales</p>
                  <v-text-field
                    solo
                    clearable
                    required
                    type="number"
                    v-model="obj.sales"
                    background-color="#e7eaf3"
                  ></v-text-field>
                </v-row>
              </v-col>
            </v-row>
          </v-form>

          <v-row>
            <v-col cols="12">
              <v-card class="pa-0">
                <v-card-text>
                  <div id="dialog" style="height: 300px; width: 100%"></div>
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </v-card-text>

      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="blue darken-1" text @click="closeDialog" id="closeModal"> Close </v-btn>
        <v-btn
          color="blue darken-1"
          text
          @click="saveElement"
          :disabled="!isFormValid"
        >
          Save
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<style>
.contenedorLabelInput {
  display: flex;
  flex-direction: column;
  gap: 2px;
}
</style>

<script>
export default {
  name: "dialogAdd",
  props: {
    dialog: Boolean,
    isEdit: Boolean,
    item: Object,
  },
  data: () => ({
    isOpenModal: false,
    obj: {
      name: null,
      direction: null,
      referens: null,
      sales: null,
    },
    formAddElement: false,
    marker: null,
  }),
  mounted() {},
  computed: {
    isFormValid() {
      return (
        this.obj.name &&
        this.obj.direction &&
        this.obj.referens &&
        this.obj.sales
      );
    },
  },
  watch: {
    item(val) {
      if (this.isEdit) {
        this.obj.id = val.id;
        this.obj.name = val.name;
        this.obj.direction = val.direction;
        this.obj.referens = val.referens;
        this.obj.sales = val.sales;
      }
    },

  

    dialog(val) {
      this.isOpenModal = val;
      if (!this.isEdit) {
        this.obj.id = null;
        this.obj.name = null;
        this.obj.direction = null;
        this.obj.referens = null;
        this.obj.sales = null;
      }else{
        this.obj.id = this.item.id;
        this.obj.name = this.item.name;
        this.obj.direction = this.item.direction;
        this.obj.referens = this.item.referens;
        this.obj.sales = this.item.sales;
      }
    },
    isOpenModal(val) {
      this.$emit("update:dialog", val);
    },
  },
  methods: {
    saveElement() {
      this.$emit("save", this.obj);
      this.closeDialog();
    },

    closeDialog() {
      this.$emit("update:dialog", false);
    },
  },
};
</script>
