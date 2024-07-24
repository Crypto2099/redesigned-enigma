<script setup>

import {onMounted, ref, watch} from "vue";
import hydra_smol from "./assets/hydra_smol.png";
import QRCodeStyling from "styled-qr-code";
import * as ed from '@noble/ed25519';
import {encode} from 'cbor-x';
// import * as cbor from 'cbor-x';

const vars = ref({
  ephemeral_key: {
    private: ed.utils.randomPrivateKey(),
    public: null,
  },
  cabinet_key: {
    private: ed.utils.randomPrivateKey(),
    public: null,
  },
  signature: null,
  payload: null,
  uri: null
});


const uri_root = `web+cardano://claim/v1?faucet_url=https%3A%2F%2Fauth.hydradoom.fun/v1&code=`;

const qr_options = ref({
  width: 512,
  height: 512,
  margin: 1,
  data: uri_root,
  image: hydra_smol,
  qrOptions: {
    typeNumber: "0",
    mode: "Byte",
    errorCorrectionLevel: "Q"
  },
  imageOptions: {
    hideBackgroundDots: false,
    imageSize: 0.8,
    margin: 0
  },
  dotsOptions: {
    type: "square",
    color: "#000000",
    // gradient: {
    //   type: "linear",
    //   rotation: -1.5707963267948966,
    //   colorStops: [{offset: 0, color: "#d6a656"}, {
    //     offset: 1,
    //     color: "#896567"
    //   }]
    // }
  },
  backgroundOptions: {color: "#ffffff"},
  dotsOptionsHelper: {
    colorType: {single: true, gradient: false},
    gradient: {
      linear: true,
      radial: false,
      color1: "#6a1a4c",
      color2: "#6a1a4c",
      rotation: "0"
    }
  },
  cornersSquareOptions: {type: "extra-rounded", color: "#000000"},
  cornersSquareOptionsHelper: {
    colorType: {
      single: true,
      gradient: false
    },
    gradient: {
      linear: true,
      radial: false,
      color1: "#000000",
      color2: "#000000",
      rotation: "0"
    }
  },
  cornersDotOptions: {type: "", color: "#000000"},
  cornersDotOptionsHelper: {
    colorType: {single: true, gradient: false},
    gradient: {
      linear: true,
      radial: false,
      color1: "#000000",
      color2: "#000000",
      rotation: "0"
    }
  },
  backgroundOptionsHelper: {
    colorType: {single: true, gradient: false},
    gradient: {
      linear: true,
      radial: false,
      color1: "#ffffff",
      color2: "#ffffff",
      rotation: "0"
    }
  }
});

const qrCode = ref(null);

const qr_code = new QRCodeStyling(qr_options.value);

onMounted(async () => {
  vars.value.cabinet_key.public = await ed.getPublicKeyAsync(vars.value.cabinet_key.private);
  await regenerate();
  qr_code.append(qrCode.value);
});

const regenerate = async () => {
  vars.value.ephemeral_key.private = ed.utils.randomPrivateKey();
  vars.value.ephemeral_key.public = await ed.getPublicKeyAsync(vars.value.ephemeral_key.private);
  vars.value.signature = await ed.signAsync(vars.value.ephemeral_key.public, vars.value.cabinet_key.private);
  vars.value.payload = ed.etc.bytesToHex(
      encode([
        vars.value.ephemeral_key.public,
        [
          vars.value.cabinet_key.public,
          vars.value.signature
        ]
      ])
  );
  vars.value.uri = uri_root + vars.value.payload;
  qr_options.value.data = vars.value.uri;
  qr_code.update(qr_options.value);
}

/*watch(qr_options, () => {
  qr_code.update(qr_options.value);
}, {
  deep: true,
});*/
</script>

<template>
  <header>
    <img alt="Hydra Logo" class="logo" src="./assets/logo.png" width="106"
         height="106" />

    <div class="wrapper">
      <h1>DOOM on Hydra: Ephemeral QR Generator</h1>
    </div>
  </header>

  <main>
    <div id="qr-code-img" ref="qrCode"></div>
  </main>
  <button type="button" @click="regenerate">
    Regenerate
  </button>
  <div id="debug" style="grid-column: 1 / span 2;">
    <h3 v-if="vars.ephemeral_key.public">
      <b>Ephemeral Key:</b> <br />
      <pre>{{ ed.etc.bytesToHex(vars.ephemeral_key.public) }}</pre>
      <br /> <b>Cabinet Key:</b> <br />
      <pre>{{ ed.etc.bytesToHex(vars.cabinet_key.public) }}</pre>
      <br /> <b>Signature:</b><br />
      <pre>{{ ed.etc.bytesToHex(vars.signature) }}</pre>
      <br /> <b>CBOR Payload:</b><br />
      <pre>{{ vars.payload }}</pre>
      <br /> <b>web+cardano URI:</b><br />
      <pre>{{ vars.uri }}</pre>
    </h3>
  </div>
</template>

<style>

button {
  background: black;
  color: white;
  border-radius: 0;
  font-size: 2rem;
  border-color: dimgray;
  padding: 0.5em;
  margin-top: 1em;
  margin-bottom: 1em;
}

#qr-code-img {
  background-color: white;
  border: 1px solid red;
  border-radius: 8px;
  color: black;
  padding: 8px;
  margin-bottom: 16px;
  margin-top: 16px;
  text-align: center;
}

pre {
  white-space: wrap;
  word-break: break-all;
}

header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
