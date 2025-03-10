<template>
  <v-stepper id="wjh-stepper" v-model="step" :mobile="isMobile">
    <v-stepper-header>
        <v-stepper-item
          v-for="currentStep in steps"
          :key="currentStep.value"
          :value="currentStep.value"
          :title="$t(`app.wjhEingabe.steps.${currentStep.value}`)"
          :aria-label="$t(`app.wjhEingabe.steps.${currentStep.value}`)"
          :icon="currentStep.icon"
          :complete-icon="currentStep.icon"
          :edit-icon="currentStep.editIcon ?? 'mdi-pencil'"
          :error="stepErrors[currentStep.value]"
          :complete="getStepNumber(currentStep.value) < stepNumber"
          :editable="getStepNumber(currentStep.value) <= maxStepNumber"
        />
    </v-stepper-header>
    <v-stepper-window>
      <v-stepper-window-item value="grunddaten">
        <v-form v-model="grunddatenValid" @submit.prevent.stop="nextStep">
        <v-container>
          <v-row>
            <v-col cols="12">
              <v-text-field
                id="familieneinkommen-field"
                ref="grunddatenFirstField"
                v-model.number="model.familieneinkommen"
                class="required"
                prepend-inner-icon="mdi-currency-eur"
                :label="$t('app.wjhEingabe.familieneinkommen.label')"
                :placeholder="$t('app.wjhEingabe.familieneinkommen.label')"
                :hint="$t('app.wjhEingabe.familieneinkommen.description')"
                type="number"
                :rules="geldBetragRules"
              >
                <template #append-inner>
                  <v-tooltip
                    v-model="showFamilieneinkommenTooltip"
                    :text="$t('app.wjhEingabe.familieneinkommen.additionalHint')"
                    :aria-label="$t('app.wjhEingabe.familieneinkommen.additionalHint')"
                    width="400px"
                    open-on-click
                    close-on-back
                    :open-on-hover="false"
                    @click:outside="showFamilieneinkommenTooltip = false"
                  >
                    <template #activator="{ props: activatorProps }">
                      <v-icon
                        v-bind="activatorProps"
                        :aria-label="$t('app.wjhEingabe.familieneinkommen.tooltipLabel')"
                        @click="showFamilieneinkommenTooltip = true"
                      >mdi-information</v-icon>
                    </template>
                  </v-tooltip>
                </template>
              </v-text-field>
            </v-col>
            <v-col cols="12">
              <v-text-field
                id="personen-field"
                v-model.number="model.personenImHaushalt"
                class="required"
                prepend-inner-icon="mdi-account-group"
                :label="$t('app.wjhEingabe.personenImHaushalt.label')"
                :placeholder="$t('app.wjhEingabe.personenImHaushalt.label')"
                type="number"
                :hint="$t('app.wjhEingabe.personenImHaushalt.description')"
                :rules="personenAnzahlRules"
              />
            </v-col>
          </v-row>
          <v-row justify="end" class="px-3">
            <v-btn id="grunddaten-next-button" type="submit" :disabled="!grunddatenValid">
              {{ $t("app.wjhEingabe.steps.weiter") }}
              <svg aria-hidden="true" class="m-button__icon">
                <use xlink:href="#icon-arrow-right"></use>
              </svg>
            </v-btn>
          </v-row>
        </v-container>
        </v-form>
      </v-stepper-window-item>
      <v-stepper-window-item value="wohnung">
        <v-form v-model="wohnungValid" @submit.prevent.stop="nextStep">
        <v-container>
          <v-row>
            <v-col cols="12">
              <v-text-field
                id="miete-field"
                ref="wohnungFirstField"
                v-model.number="model.miete"
                class="required"
                prepend-inner-icon="mdi-currency-eur"
                :label="$t('app.wjhEingabe.miete.label')"
                :placeholder="$t('app.wjhEingabe.miete.label')"
                type="number"
                :hint="$t('app.wjhEingabe.miete.description')"
                :rules="geldBetragRules"
              />
            </v-col>
            <v-col cols="12">
              <v-text-field
                id="groesse-wohnung-field"
                v-model.number="model.groesseWohnung"
                class="required"
                prepend-inner-icon="mdi-vector-square"
                :label="$t('app.wjhEingabe.groesseWohnung')"
                :placeholder="$t('app.wjhEingabe.groesseWohnung')"
                type="number"
                :hint="nebenkostenText"
                :rules="wohungsgroesseRules"
              />
            </v-col>
            <v-col v-if="mietobergrenze < (model.miete ?? 0)" cols="12">
              <v-alert type="info" color="primary">
                <b>{{ $t("app.wjhEingabe.mietobergrenze.label") }}: </b>
                <span>{{ mietobergrenze }}€</span>
                <br />
                <span>
                  {{ $t("app.wjhEingabe.mietobergrenze.description") }}
                </span>
                <br/>
              </v-alert>
            </v-col>
            <v-col cols="12">
              <span class="m-label">
                {{ $t("app.wjhEingabe.kostenWohnungGesamt.label") }}: {{ verwendeteMiete }}€
                <v-tooltip
                    v-model="showVerwendeteMieteTooltip"
                    :text="wohnkostenGesamtBerechnungsText"
                    :aria-label="$t('app.wjhEingabe.kostenWohnungGesamt.additionalHint')"
                    open-on-click
                    close-on-back
                    :open-on-hover="false"
                    @click:outside="showVerwendeteMieteTooltip = false"
                  >
                    <template #activator="{ props: activatorProps }">
                      <v-icon
                        v-bind="activatorProps"
                        :aria-label="$t('app.wjhEingabe.kostenWohnungGesamt.additionalHint')"
                        @click="showVerwendeteMieteTooltip = true"
                      >mdi-information</v-icon>
                    </template>
                  </v-tooltip>
              </span>
              <span class="m-label">
                {{ $t("app.wjhEingabe.einkommensgrenze.label") }}: {{ einkommensgrenze }}€
                <v-tooltip
                    v-model="showEinkommensgrenzeTooltip"
                    :text="einkommensgrenzeBerechnungstext"
                    :aria-label="$t('app.wjhEingabe.einkommensgrenze.additionalHint')"
                    open-on-click
                    close-on-back
                    :open-on-hover="false"
                    @click:outside="showEinkommensgrenzeTooltip = false"
                  >
                    <template #activator="{ props: activatorProps }">
                      <v-icon
                        v-bind="activatorProps"
                        :aria-label="$t('app.wjhEingabe.einkommensgrenze.additionalHint')"
                        @click="showEinkommensgrenzeTooltip = true"
                      >mdi-information</v-icon>
                    </template>
                  </v-tooltip>
              </span>
              <span id="uebersteigendes-einkommen" class="m-label">{{ $t("app.wjhEingabe.uebersteigendesEinkommen") }}: {{ uebersteigendesEinkommen }}€</span>
            </v-col>
          </v-row>
          <v-row justify="space-between" class="px-3">
            <v-btn id="wohnung-back-button" @click="step='grunddaten'">
              <svg aria-hidden="true" class="m-button__icon ml-0 mr-2">
                <use xlink:href="#icon-arrow-left"></use>
              </svg>
              {{ $t("app.wjhEingabe.steps.zurueck") }}
            </v-btn>
            <v-btn id="wohnung-next-button" type="submit" :disabled="!wohnungValid">
              {{ $t("app.wjhEingabe.steps.weiter") }}
              <svg aria-hidden="true" class="m-button__icon">
                <use xlink:href="#icon-arrow-right"></use>
              </svg>
            </v-btn>
          </v-row>
        </v-container>
        </v-form>
      </v-stepper-window-item>
      <v-stepper-window-item value="kitakosten">
        <v-form v-model="kitakostenValid" @submit.prevent.stop="nextStep">
        <v-container>
          <v-row>
            <v-col cols="12">
              <v-text-field
                id="kitakosten-geschwister-field"
                ref="kitakostenFirstField"
                v-model.number="model.kitaKostenGeschwister"
                class="required"
                prepend-inner-icon="mdi-currency-eur"
                :label="$t('app.wjhEingabe.kitaKostenGeschwister.label')"
                :placeholder="$t('app.wjhEingabe.kitaKostenGeschwister.label')"
                type="number"
                :hint="$t('app.wjhEingabe.kitaKostenGeschwister.description')"
                :rules="geldBetragRules"
              >
                <template #append-inner>
                  <v-tooltip
                    v-model="showKitakostenGeschwisterTooltip"
                    :text="$t('app.wjhEingabe.kitaKostenGeschwister.additionalHint')"
                    :aria-label="$t('app.wjhEingabe.kitaKostenGeschwister.additionalHint')"
                    width="400px"
                    open-on-click
                    close-on-back
                    :open-on-hover="false"
                    @click:outside="showKitakostenGeschwisterTooltip = false"
                  >
                    <template #activator="{ props: activatorProps }">
                      <v-icon
                        v-bind="activatorProps"
                        :aria-label="$t('app.wjhEingabe.kitaKostenGeschwister.tooltipLabel')"
                        @click="showKitakostenGeschwisterTooltip = true"
                      >mdi-information</v-icon>
                    </template>
                  </v-tooltip>
                </template>
              </v-text-field>
            </v-col>
            <v-col cols="12">
              <v-alert type="info" color="primary">
                <b>{{ $t("app.wjhEingabe.uebersteigendesEinkommen") }}: </b>
                <span>{{ uebersteigendesEinkommenMinusGeschwister }}€</span>
                <div v-if="uebersteigendesEinkommenMinusGeschwister > 0">
                  <b>{{ $t("app.wjhEingabe.eigenanteil.label") }}: </b>
                  <span id="eigenanteil">{{ eigenanteil }}€</span>
                  <br />
                  <span>
                    {{ $t("app.wjhEingabe.eigenanteil.description") }}
                  </span>
                </div>
              </v-alert>
            </v-col>
          </v-row>
          <v-row justify="space-between" class="px-3">
            <v-btn id="kitakosten-back-button" @click="step='wohnung'">
              <svg aria-hidden="true" class="m-button__icon ml-0 mr-2">
                <use xlink:href="#icon-arrow-left"></use>
              </svg>
              {{ $t("app.wjhEingabe.steps.zurueck") }}
            </v-btn>
            <v-btn id="kitakosten-next-button" type="submit" :disabled="!kitakostenValid">
              {{ $t("app.wjhEingabe.steps.weiter") }}
              <svg aria-hidden="true" class="m-button__icon">
                <use xlink:href="#icon-arrow-right"></use>
              </svg>
            </v-btn>
          </v-row>
        </v-container>
        </v-form>
      </v-stepper-window-item>
      <v-stepper-window-item value="ergebnis">
        <v-form v-model="ergebnisValid" @submit.prevent.stop="nextStep">
        <v-container>
          <v-row>
            <v-col cols="12" class="py-0">
              <h3 class="m-label">{{ $t("app.wjhErgebnis.title") }}</h3>
              <span class="m-hint">{{ $t("app.wjhErgebnis.hint") }}</span>
            </v-col>
            <v-col v-if="volleFoerderung">
              <v-alert type="success">
                {{ $t("app.wjhErgebnis.volleFoerderung") }}
              </v-alert>
            </v-col>
            <v-col v-else cols="12">
              <v-alert type="info" color="primary">
                <span>{{ $t("app.wjhErgebnis.ueberEinkommensgrenze") }} {{ foerderung ? $t("app.wjhErgebnis.teilfoerderung") : '' }}</span>
                <br />
                <b>{{ $t("app.wjhEingabe.uebersteigendesEinkommen") }}: </b>
                <span>{{ uebersteigendesEinkommenMinusGeschwister }}€</span>
                <br />
                <b>{{ $t("app.wjhEingabe.eigenanteil.label") }}: </b>
                <span id="ergebnis-eigenanteil">{{ eigenanteil }}€</span>
                <br />
                <span>
                  {{ $t("app.wjhEingabe.eigenanteil.description") }}
                </span>
              </v-alert>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12">
              <v-text-field
                id="kitakosten-field"
                ref="ergebnisFirstField"
                v-model.number="model.kitaKosten"
                class="required"
                prepend-inner-icon="mdi-currency-eur"
                :label="$t('app.wjhEingabe.kitaKosten.label')"
                :placeholder="$t('app.wjhEingabe.kitaKosten.label')"
                :hint="$t('app.wjhEingabe.kitaKosten.description')"
                type="number"
                :rules="geldBetragRules"
              />
            </v-col>
            <v-col cols="12">
              <v-progress-linear
                :max="model.kitaKosten"
                :model-value="nichtGefoerderterBetrag"
                :aria-label="$t('app.wjhErgebnis.progressBarTooltip')"
                height="10em"
                bg-color="success"
                color="error"
                reverse
                bg-opacity="1"
              />
            </v-col>
          </v-row>
          <v-row justify="space-between">
            <v-col cols="auto">
              <span id="ergebnis-foerderung" class="m-label">{{ $t("app.wjhErgebnis.foerderung") }}: {{ foerderung }}€</span>
            </v-col>
            <v-col cols="auto">
              <span id="ergebnis-nicht-gefoerdert" class="m-label">{{ $t("app.wjhErgebnis.eigenanteil") }}: {{ nichtGefoerderterBetrag }}€</span>
            </v-col>
          </v-row>
          <v-row justify="space-between" class="px-3">
            <v-btn id="result-back-button" @click="resultBack">
              <svg aria-hidden="true" class="m-button__icon ml-0 mr-2">
                <use xlink:href="#icon-arrow-left"></use>
              </svg>
              {{ $t("app.wjhEingabe.steps.zurueck") }}
            </v-btn>
          </v-row>
        </v-container>
        </v-form>
      </v-stepper-window-item>
    </v-stepper-window>
  </v-stepper>
</template>

<script setup lang="ts">
import { defineModel, ref, computed, watch, nextTick  } from 'vue'
import { UserData } from '@/api/wjhTypes'
import { getGrundbetragMitFamilie, getMietobergrenze, nebenkostenProQm, grundbetrag, familienzuschlag } from '@/constants'
import { useI18n } from "vue-i18n";
const { t } = useI18n({ useScope: "global" });

// Verwaltung des aktiven Schrittes im stepper
const steps = [
  { value: "grunddaten", icon: "mdi-cash", editIcon: "mdi-cash-edit" },
  { value: "wohnung", icon: "mdi-home", editIcon: "mdi-home-edit" },
  { value: "kitakosten", icon: "mdi-cash", editIcon: "mdi-cash-edit" },
  { value: "ergebnis", icon: "mdi-information", editIcon: "mdi-check" }
]
const step = ref("grunddaten")
const getStepNumber = (step : string) => {
  return steps.findIndex(s => s.value == step);
}
const stepNumber = computed(() : number => {
  return getStepNumber(step.value)
})
let maxStepNumber = stepNumber.value;
// maxStepNumber setzen
watch(stepNumber, (newValue : number) => {
  maxStepNumber = Math.max(newValue, maxStepNumber);
})

const nextStep = () => {
  switch(step.value) {
    case "grunddaten": {
      grunddatenNext()
      return
    }
    case "wohnung": {
      wohnungNext()
      return
    }
    case "kitakosten": {
      kitakostenNext()
      return
    }
  }
}

// Funktion, die vom Schritt "grunddaten" aus weiter springt.
const grunddatenNext = () => {
  if(grunddatenValid.value) {
    if(!grundbetragAusreichend.value) {
      step.value = "wohnung";
    } else {
      step.value = "ergebnis";
    }
  }
}
// Funktion, die vom Schritt "wohnung" aus weiter springt.
const wohnungNext = () => {
  if (wohnungValid.value) {
    if (uebersteigendesEinkommen.value) {
      step.value = "kitakosten";
    } else {
      step.value = "ergebnis"
    }
  }
}
// Funktion, die vom Schritt "kitakosten" aus weiter springt.
const kitakostenNext = () => {
  if (kitakostenValid.value) {
    step.value = "ergebnis";
  }
}
// Funktion, die vom Schritt "ergebnis" aus zurück springt.
const resultBack = () => {
  if (grundbetragAusreichend.value) {
    step.value = "grunddaten"
  } else if (!uebersteigendesEinkommen.value) {
    // kein übersteigendes einkommen nach Schritt Wohnung
    step.value = "wohnung"
  } else {
    step.value = "kitakosten"
  }
}

// In der Eingabemaske verwendete Daten.
const model = defineModel<UserData>({ default: {}})
const props = defineProps({
  // gibt an, ob die Anzeige für mobile Geräte optimiert werden soll
  isMobile: {
    type: Boolean,
    default: false
  },
  // Gibt an, ob die Komponente bereits aktiv ist
  isActive: {
    type: Boolean,
    default: true
  }
});

// select() auf erstes Feld
const grunddatenFirstField = ref<HTMLInputElement|null>(null)
const wohnungFirstField = ref<HTMLInputElement|null>(null)
const kitakostenFirstField = ref<HTMLInputElement|null>(null)
const ergebnisFirstField = ref<HTMLInputElement|null>(null)
const selectFirstInputInStep = (stepToSelect : String|undefined) => {
  if (!stepToSelect) {
    stepToSelect = step.value
  }
  nextTick(() => { 
    switch(stepToSelect) {
      case "grunddaten":
        if (grunddatenFirstField.value) {
          grunddatenFirstField.value.select()
        }
        break;
      case "wohnung":
        if (wohnungFirstField.value) {
          wohnungFirstField.value.select()
        }
        break;
      case "kitakosten":
        if (kitakostenFirstField.value) {
          kitakostenFirstField.value.select()
        }
        break;
      case "ergebnis":
        if (ergebnisFirstField.value) {
          ergebnisFirstField.value.select()
        }
        break;
      default:
        return;
    }
  })
}
watch(step, (newValue : String) => {
  selectFirstInputInStep(newValue)
})
watch(() => props.isActive, (newValue : boolean) => {
  if (newValue) {
    selectFirstInputInStep(step.value)
  }
})

const showFamilieneinkommenTooltip = ref(false);
const showEinkommensgrenzeTooltip = ref(false);
const showVerwendeteMieteTooltip = ref(false);
const showKitakostenGeschwisterTooltip = ref(false);

// Grundbetrag der Einkommensgrenze inklusive des Familianzuschlags.
const grundbetragMitFamilie = computed(() => {
  return getGrundbetragMitFamilie(model.value.personenImHaushalt ?? 1);
})

// Obergrenze der Miete, die in die Einkommensgrenze einfließen kann.
const mietobergrenze = computed(() => {
  const mietobergrenze = getMietobergrenze(model.value.personenImHaushalt ?? 1);
  return mietobergrenze?.miete ?? 0;
})

// Nebenkosten, anhand der angegebenen Fläche
const nebenkostenWohnung = computed(() => {
  return Math.round((model.value.groesseWohnung ?? 0) * nebenkostenProQm);
})

// Nebenkosten, anhand der angegebenen Fläche
const nebenkostenText = computed(() => {
  return t("app.wjhEingabe.nebenkostenWohnung.label", [nebenkostenWohnung.value]);
})

// Berechnung der wohnkosten
const wohnkostenGesamtBerechnungsText = computed(() => {
  return `${t("app.wjhEingabe.miete.shortLabel")}: ${mieteMitObergrenze.value}€ + ${t("app.wjhEingabe.nebenkostenWohnung.shortLabel")}: ${nebenkostenWohnung.value}€`;
})

// Berechnung der einkommensgrenze
const einkommensgrenzeBerechnungstext = computed(() => {
  const mieteText = `${t("app.wjhEingabe.miete.shortLabel")}: ${verwendeteMiete.value}€`
  const grundbetragText = `${t("app.wjhEingabe.grundbetrag")}: ${grundbetrag}€`

  const zusaetzlichePersonen = model.value.personenImHaushalt ? model.value.personenImHaushalt-1 : 0
  const familienzuschlagText = `${zusaetzlichePersonen}*(${t("app.wjhEingabe.familienzuschlag")}: ${familienzuschlag}€)`

  return zusaetzlichePersonen ? `${mieteText} + ${grundbetragText} + ${familienzuschlagText}` : `${mieteText} + ${grundbetragText}`;
})

// Für die Miete tatsächlich verwendeter Wert unter Berücksichtigung der Mietobergrenze
const mieteMitObergrenze = computed(() => {
  return Math.min(mietobergrenze.value, model.value.miete ?? 0);
})

// Für die Wohnkosten tatsächlich verwendeter Wert
const verwendeteMiete = computed(() => {
  return mieteMitObergrenze.value + nebenkostenWohnung.value;
})

// Grenze des Einkommens, das nicht für Kita-Kosten belastet wird.
const einkommensgrenze = computed(() => {
  return verwendeteMiete.value + grundbetragMitFamilie.value;
})

// Anteil des Einkommens, der die Einkommensgrenze überschreitet.
const uebersteigendesEinkommen = computed(() => {
  return Math.max(0, (model.value.familieneinkommen ?? 0) - einkommensgrenze.value);
})

// Anteil des Einkommens, der die Einkommensgrenze überschreitet und nicht schon für andere Kita-Kosten verwendet wird.
const uebersteigendesEinkommenMinusGeschwister = computed(() => {
  return Math.max(0, (model.value.familieneinkommen ?? 0) - einkommensgrenze.value - (model.value.kitaKostenGeschwister ?? 0));
})

// Anteil des Einkommens, der für die Kita-Kosten belastet wird.
const eigenanteil = computed(() => {
  return Math.round(uebersteigendesEinkommenMinusGeschwister.value * 0.3);
})

// Anteil der Kitakosten, die vorraussichtlich selbst gezahlt werden müssen.
const nichtGefoerderterBetrag = computed(() => {
  return Math.min(eigenanteil.value, (model.value.kitaKosten ?? 0));
})
// Anteil der Kitakosten, der vorraussichtlich gefördert wird.
const foerderung = computed(() => {
  return Math.max(0, (model.value.kitaKosten ?? 0) - nichtGefoerderterBetrag.value);
})

// Status-Felder
// Gibt an, ob der grundbetragMitFamilie bereits hoch genug ist um eine komplette Förderung zu erhalten.
const grundbetragAusreichend = computed(() => {
  return grundbetragMitFamilie.value >= (model.value.familieneinkommen ?? 0);
})

// Gibt an, ob vorraussichtlich mit einer vollen Förderung gerechnet werden kann.
const volleFoerderung = computed(() => {
  return uebersteigendesEinkommenMinusGeschwister.value <= 0;
})

// Validierung
const grunddatenValid = ref(true);
const wohnungValid = ref(true);
const ergebnisValid = ref(true);
const kitakostenValid = ref(true);

const stepErrors = computed<{[key: string]: boolean}>(() => {
  return {
    "grunddaten": !grunddatenValid.value,
    "wohnung": !wohnungValid.value,
    "kitakosten": !kitakostenValid.value,
    "ergebnis": false
  }
});

const geldBetragRules = [
  (v : number) => !!v || v === 0 || t("validation.zahl"),
  (v : number) => v >= 0 || t("validation.positiveZahl"),
  (v : number) => v <= 1000000 || t("validation.maximalbetrag")
]

const wohungsgroesseRules = [
  (v : number) => !!v || v === 0 || t("validation.zahl"),
  (v : number) => v >= 0 || t("validation.positiveZahl"),
  (v : number) => v <= 1000 || t("validation.zuGross")
]

const personenAnzahlRules = [
  (v : number) => !!v || v === 0 || t("validation.zahl"),
  (v : number) => v >= 1 || t("validation.mindestensEins"),
  (v : number) => v <= 100 || t("validation.zuGross")
]
</script>

<style scoped>
.v-window {
  margin-left: 0;
  margin-right: 0;
}

#wjh-stepper {
  color: #3A5368;
}
</style>