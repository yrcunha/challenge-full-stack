<script>
import api from "../data/api";
import Button from "@/components/Button";
import BaseView from "@/components/Base";
import {configHeaders} from "@/data/config-headers";
import {getValue} from "@/data/local-storage";

export default {
  name: "RegisterStudent",
  components: { Button, BaseView },
  data: () => ({
    buttons: ["Cancelar", "Salvar"],
    fields: ["name", "email", "cpf"],
    model: {
      name: "",
      email: "",
      cpf: "",
    },
    rules: {
      name: [(v) => !!v || "Nome é obrigatório"],
      email: [
        (v) => !!v || "E-mail é obrigatório",
        (v) => /.+@.+\..+/.test(v) || "E-mail inválido",
      ],
      cpf: [
        (v) => !!v || "CPF é obrigatório",
        (v) => v.split('').length === 11 || "CPF inválido",
      ],
    },
    valid: false,
    showSnackbar: false,
    snackbarTimeout: 5000,
    snackbarText: "",
    snackbarColor: "",
  }),
  methods: {
    validateForm() {
      this.$refs.form.validate();
    },
    resetForm() {
      this.model.name = "";
      this.model.email = "";
      this.model.cpf = "";
      this.$refs.form.resetValidation();
    },
    getFormData() {
      return {
        name: this.model.name,
        email: this.model.email,
        cpf: this.model.cpf,
      };
    },
    submitForm() {
      this.validateForm();
      if (this.valid) {
        const formData = this.getFormData();
        this.saveStudent(formData);
      }
    },
    async saveStudent(formData) {
      try {
        const response = await api.post("/students", formData, configHeaders(getValue("token")));
        this.callSnackBar(response.data.message);
        this.resetForm();
      } catch (error) {
        this.callSnackBar(error.response.data.code, "error");
      }
    },
    callSnackBar(text = "Registro realizado com sucesso!", status = "success") {
      status === "error"
        ? (this.snackbarColor = "red accent-2")
        : (this.snackbarColor = "green");
      this.snackbarText = text;
      this.showSnackbar = true;
    },
  },
};
</script>

<template>
  <div>
    <v-app-bar app>
      <div>Cadastrar aluno</div>
    </v-app-bar>
    <BaseView />
    <v-main>
      <v-container class="mt-3">
        <v-col class="col-6 mx-auto">
          <v-form ref="form" v-model="valid">
            <v-row v-for="field in fields" :key="field">
              <v-col>
                <v-text-field
                  v-model="model[field]"
                  :label="field.toUpperCase()"
                  :rules="rules[field]"
                  outlined
                  required
                  :hide-spin-buttons="field === 'cpf'"
                  :type="field === 'cpf' ? 'number' : undefined"
                ></v-text-field>
              </v-col>
            </v-row>
            <v-row justify="center">
              <v-col class="col-auto" v-for="button in buttons" :key="button">
                <Button
                  :context_botton="button.toUpperCase()"
                  :color_botton="button === 'Salvar' ? 'green' : 'red'"
                  class_botton="white--text"
                  :redirect_botton="
                    button === 'Cancelar' ? '/students' : undefined
                  "
                  :action_click="button === 'Salvar' ? submitForm : undefined"
                />
              </v-col>
            </v-row>
          </v-form>
        </v-col>
      </v-container>
      <v-snackbar
        v-model="showSnackbar"
        :timeout="snackbarTimeout"
        :color="snackbarColor"
        >{{ snackbarText }}</v-snackbar
      >
    </v-main>
  </div>
</template>
