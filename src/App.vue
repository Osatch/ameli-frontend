<template>
  <div class="container">
    <h1>Extraction Ameli 🩺</h1>

    <form @submit.prevent="envoyerFormulaire">

      <label for="profession">Profession :</label>
      <input
        list="professions"
        v-model="form.profession"
        placeholder="Ex: Cardiologue"
        required
      />
      <datalist id="professions">
        <option v-for="p in professions" :key="p" :value="p" />
      </datalist>

      <div v-for="(dep, index) in form.departements" :key="index" class="dep-row">
        <input
          list="departements"
          v-model="form.departements[index]"
          placeholder="Ex: 13"
          required
        />
        <button type="button" @click="retirerDepartement(index)">❌</button>
      </div>

      <button type="button" @click="ajouterDepartement">➕ Ajouter un département</button>

      <datalist id="departements">
        <option v-for="d in departements" :key="d.code" :value="d.code">
          {{ d.nom }}
        </option>
      </datalist>

      <label for="option">Type de numéros :</label>
      <select v-model="form.option" required>
        <option :value="1">Tous les numéros</option>
        <option :value="2">Numéros 06/07 uniquement</option>
      </select>

      <button type="submit">Lancer l'extraction</button>
    </form>

    <div v-if="message" class="message">{{ message }}</div>

    <div v-if="fichierUrl" class="telechargement">
      <a :href="fichierUrl" target="_blank" download>Télécharger le fichier Excel 📄</a>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      form: {
        profession: '',
        departements: [''],
        option: 1
      },
      professions: [
        "Masseur-kinésithérapeute", "Gynécologues / Obstétricien", "Infirmier", "Infirmier en pratique avancée",
        "Ophtalmologiste", "Chirurgiens-dentistes", "Médecin généraliste", "Acupuncteur", "Allergologue",
        "Ambulance / Véhicule sanitaire léger", "Anatomo-Cyto-Pathologiste", "Anesthésiste réanimateur",
        "Angiologue", "Cancérologues", "Cardiologue", "Chirurgien-dentiste spécialiste en orthopédie dento-faciale",
        "Chirurgien général", "Chirurgien infantile", "Chirurgien maxillo-facial", "Chirurgien maxillo-facial et stomatologiste",
        "Chirurgien oral", "Chirurgien orthopédiste et traumatologue", "Chirurgien plasticien", "Chirurgien thoracique et cardio-vasculaire",
        "Chirurgien urologue", "Chirurgien vasculaire", "Chirurgien viscéral", "Dermatologue et vénérologue", "Echographiste",
        "Endocrinologue-diabétologue", "Fournisseur de matériel médical et para-médical", "Gastro-entérologue et hépatologue",
        "Gériatre", "Hématologue", "Homéopathe", "Laboratoires", "Médecin biologiste", "Médecin généticien",
        "Médecin spécialiste en médecine nucléaire", "Médecin spécialiste en santé publique et médecine sociale",
        "Médecin thermaliste", "Médecine appliquée aux sports", "Médecine d'urgence", "Médecine des Maladies infectieuses et tropicales",
        "Médecine légale et expertises médicales", "Médecine vasculaire", "Néphrologue", "Neurochirurgien", "Neurologue",
        "Neuropsychiatre", "Orthophoniste", "Orthoptiste", "Oto-Rhino-Laryngologue (ORL) et chirurgien cervico-facial",
        "Pédiatre", "Pédicure-podologue", "Pharmacien", "Phoniatre", "Pneumologue", "Psychiatres", "Radiologue",
        "Radiothérapeute", "Réanimateur médical", "Rhumatologue", "Sage-femme", "Spécialiste en allergologie",
        "Spécialiste en médecine interne", "Spécialiste en médecine physique et de réadaptation", "Stomatologistes"
      ],
      departements: [
        { code: "01", nom: "Ain" }, { code: "02", nom: "Aisne" }, { code: "2A", nom: "Corse-du-Sud" }, { code: "2B", nom: "Haute-Corse" },
        { code: "03", nom: "Allier" }, { code: "04", nom: "Alpes-de-Haute-Provence" }, { code: "05", nom: "Hautes-Alpes" },
        { code: "06", nom: "Alpes-Maritimes" }, { code: "07", nom: "Ardèche" }, { code: "08", nom: "Ardennes" },
        { code: "09", nom: "Ariège" }, { code: "10", nom: "Aube" }, { code: "11", nom: "Aude" }, { code: "12", nom: "Aveyron" },
        { code: "13", nom: "Bouches-du-Rhône" }, { code: "14", nom: "Calvados" }, { code: "15", nom: "Cantal" },
        { code: "16", nom: "Charente" }, { code: "17", nom: "Charente-Maritime" }, { code: "18", nom: "Cher" },
        { code: "19", nom: "Corrèze" }, { code: "21", nom: "Côte-d'Or" }, { code: "22", nom: "Côtes-d'Armor" },
        { code: "23", nom: "Creuse" }, { code: "24", nom: "Dordogne" }, { code: "25", nom: "Doubs" },
        { code: "26", nom: "Drôme" }, { code: "27", nom: "Eure" }, { code: "28", nom: "Eure-et-Loir" },
        { code: "29", nom: "Finistère" }, { code: "30", nom: "Gard" }, { code: "31", nom: "Haute-Garonne" },
        { code: "32", nom: "Gers" }, { code: "33", nom: "Gironde" }, { code: "34", nom: "Hérault" },
        { code: "35", nom: "Ille-et-Vilaine" }, { code: "36", nom: "Indre" }, { code: "37", nom: "Indre-et-Loire" },
        { code: "38", nom: "Isère" }, { code: "39", nom: "Jura" }, { code: "40", nom: "Landes" },
        { code: "41", nom: "Loir-et-Cher" }, { code: "42", nom: "Loire" }, { code: "43", nom: "Haute-Loire" },
        { code: "44", nom: "Loire-Atlantique" }, { code: "45", nom: "Loiret" }, { code: "46", nom: "Lot" },
        { code: "47", nom: "Lot-et-Garonne" }, { code: "48", nom: "Lozère" }, { code: "49", nom: "Maine-et-Loire" },
        { code: "50", nom: "Manche" }, { code: "51", nom: "Marne" }, { code: "52", nom: "Haute-Marne" },
        { code: "53", nom: "Mayenne" }, { code: "54", nom: "Meurthe-et-Moselle" }, { code: "55", nom: "Meuse" },
        { code: "56", nom: "Morbihan" }, { code: "57", nom: "Moselle" }, { code: "58", nom: "Nièvre" },
        { code: "59", nom: "Nord" }, { code: "60", nom: "Oise" }, { code: "61", nom: "Orne" }, { code: "62", nom: "Pas-de-Calais" },
        { code: "63", nom: "Puy-de-Dôme" }, { code: "64", nom: "Pyrénées-Atlantiques" }, { code: "65", nom: "Hautes-Pyrénées" },
        { code: "66", nom: "Pyrénées-Orientales" }, { code: "67", nom: "Bas-Rhin" }, { code: "68", nom: "Haut-Rhin" },
        { code: "69", nom: "Rhône" }, { code: "70", nom: "Haute-Saône" }, { code: "71", nom: "Saône-et-Loire" },
        { code: "72", nom: "Sarthe" }, { code: "73", nom: "Savoie" }, { code: "74", nom: "Haute-Savoie" },
        { code: "75", nom: "Paris" }, { code: "76", nom: "Seine-Maritime" }, { code: "77", nom: "Seine-et-Marne" },
        { code: "78", nom: "Yvelines" }, { code: "79", nom: "Deux-Sèvres" }, { code: "80", nom: "Somme" },
        { code: "81", nom: "Tarn" }, { code: "82", nom: "Tarn-et-Garonne" }, { code: "83", nom: "Var" },
        { code: "84", nom: "Vaucluse" }, { code: "85", nom: "Vendée" }, { code: "86", nom: "Vienne" },
        { code: "87", nom: "Haute-Vienne" }, { code: "88", nom: "Vosges" }, { code: "89", nom: "Yonne" },
        { code: "90", nom: "Territoire de Belfort" }, { code: "91", nom: "Essonne" }, { code: "92", nom: "Hauts-de-Seine" },
        { code: "93", nom: "Seine-Saint-Denis" }, { code: "94", nom: "Val-de-Marne" }, { code: "95", nom: "Val-d'Oise" },
        { code: "971", nom: "Guadeloupe" }, { code: "972", nom: "Martinique" }, { code: "973", nom: "Guyane" },
        { code: "974", nom: "La Réunion" }, { code: "976", nom: "Mayotte" }
      ],
      message: '',
      fichierUrl: ''
    };
  },
  methods: {
    ajouterDepartement() {
      this.form.departements.push('');
    },
    retirerDepartement(index) {
      this.form.departements.splice(index, 1);
    },
    async envoyerFormulaire() {
      try {
        const response = await fetch("http://127.0.0.1:8000/api/launch/", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(this.form)
        });

        const result = await response.json();

        if (result.status === "success" && result.url) {
          this.message = "✅ Extraction terminée.";
          this.fichierUrl = result.url;
        } else {
          this.message = result.message || "Erreur inconnue.";
          this.fichierUrl = '';
        }
      } catch (error) {
        this.message = "Erreur lors de l’envoi du formulaire.";
        this.fichierUrl = '';
        console.error(error);
      }
    }
  }
};
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: auto;
  font-family: Arial;
}
label {
  display: block;
  margin-top: 1em;
}
input, select {
  width: 100%;
  padding: 8px;
  margin-top: 4px;
}
.dep-row {
  display: flex;
  gap: 10px;
  margin-top: 8px;
}
button {
  margin-top: 10px;
  padding: 8px 12px;
}
.message {
  margin-top: 20px;
  background: #def;
  padding: 10px;
}
.telechargement {
  margin-top: 15px;
  padding: 10px;
  background: #e6ffe6;
}
.telechargement a {
  color: #0a5c0a;
  font-weight: bold;
  text-decoration: underline;
}
</style>
