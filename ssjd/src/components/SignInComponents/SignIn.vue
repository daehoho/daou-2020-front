<template>
  <v-card class="px-4">
    <v-card-text>
      <v-form
        ref="form"
        v-model="valid"
        lazy-validation
        @submit.prevent="logIn"
      >
        <v-text-field
          autofocus
          v-model="id"
          :counter="20"
          :rules="idRules"
          label="ID"
          required
        ></v-text-field>
        <v-text-field
          type="password"
          v-model="password"
          :rules="pwdRules"
          label="Password"
          required
        ></v-text-field>
        <v-btn :disabled="!valid" class="mr-4" type="submit">Sign in</v-btn>
      </v-form>
    </v-card-text>
  </v-card>
</template>

<script>
import crypto from "crypto";

export default {
  data() {
    return {
      id: "",
      password: "",
      idRules: [
        (v) => !!v || "ID is required",
        (v) =>
          (v && v.length <= 20) || "Your ID must be less than 20 characters",
      ],
      pwdRules: [
        (v) => !!v || "Enter the password",
        (v) =>
          (v && v.length <= 20) ||
          "Your password must be less than 20 characters",
      ],
      valid: false,
      disabled: false,
    };
  },
  methods: {
    logIn() {
      const nickname = this.id;
      const salt = nickname.concat(
        nickname.slice(2, nickname.length - 2),
        nickname
          .split("")
          .reverse()
          .join("")
          .slice(0, nickname.length - 1),
        nickname.slice(3, nickname.length - 3)
      );
      const password = crypto
        .pbkdf2Sync(this.password, salt, 1038, 64, "sha512")
        .toString("base64")
        .replace(/=/gi, "");
      this.$store
        .dispatch("LOGIN", { nickname, password })
        .then(() => this.redirect())
        .catch(() => {
          alert("아이디 혹은 비밀번호가 일치하지 않습니다!");
        });
    },
    redirect() {
      const { search } = window.location;
      // console.log(`serch: ${search}`);
      const tokens = search.replace(/^\?/, "").split("&");
      // console.log(`tokens: ${tokens}`);
      const { returnPath } = tokens.reduce((qs, tkn) => {
        // console.log(`qs: ${qs}, tkn: ${tkn}`);
        const pair = tkn.split("=");
        // console.log(`pair: ${pair}`);
        qs[pair[0]] = decodeURIComponent(pair[1]);
        // console.log(`qs pair: ${qs[pair[0]]}`);
        return qs;
      }, {});
      // console.log(`returPath: ${returnPath}`);

      if (returnPath === undefined) this.$router.push("/");
      else this.$router.push(returnPath);
    },
  },
};
</script>