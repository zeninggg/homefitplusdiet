<template>
  <div
    class="accordion md-accordion accordion-3 z-depth-1-half"
    id="accordionEx194"
    role="tablist"
    aria-multiselectable="true"
  >
    <ul class="justify-content-center pt-5">
      <i class="fas fa-dumbbell mr-3 fa-2x amber-text"></i>
      <i class="fas fa-diagnoses mr-3 fa-2x orange-text"></i>
      <i class="fas fa-running mr-3 fa-2x deep-orange-text"></i>
    </ul>

    <h2
      id="title"
      class="text-center text-uppercase warning-color-text py-4 px-3"
    >
      Here are your workouts for the day!
    </h2>
    <button
      v-on:click="tracker(intensity, totalCaloriesBurnt)"
      class="btn black btn"
      style="color: white"
    >
      COMPLETED AND TRACK!
    </button>
    <h3 id="title" v-if="this.intensity == 'light'">
      Do 5 Sets of 10 for each of the exercises below. <br />
      Rest for 60 seconds in between each set. <br />Expected duration: 30
      minutes - Total Calories Burned: {{ totalCaloriesBurnt }}
    </h3>
    <h3 id="title" v-if="this.intensity == 'moderate'">
      Do 7 Sets of 12 for each of the exercises below. <br />
      Rest for 75 seconds in between each set. <br />Expected duration: 45
      minutes - Total Calories Burned: {{ totalCaloriesBurnt }}
    </h3>
    <h3 id="title" v-if="this.intensity == 'intense'">
      Do 9 Sets of 15 for each of the exercises below. <br />
      Rest for 90 seconds in between each set. <br />Expected duration: 60
      minutes - Total Calories Burned: {{ totalCaloriesBurnt }}
    </h3>

    <hr class="mb-0" />

    <div class="rgba-black-slight py-5 px-4">
      <div class="row d-flex justify-content-center">
        <div class="col-md-10 col-xl-8">
          <!--Accordion wrapper-->
          <div
            class="accordion md-accordion accordion-5"
            id="accordionEx5"
            role="tablist"
            aria-multiselectable="true"
          >
            <!-- Accordion card -->
            <div
              class="card mb-4"
              v-for="(exercise, index) in exercises"
              v-bind:key="index"
            >
              <!-- Card header -->
              <div
                class="card-header elegant-color accent-1"
                role="tab"
                id="heading33"
              >
                <a
                  data-toggle="collapse"
                  data-parent="#accordionEx5"
                  href="#collapse30"
                  aria-expanded="true"
                  aria-controls="collapse30"
                >
                  <h4
                    class="justify-content-center text-uppercase white-text mb-0 py-3 mt-1"
                  >
                    {{ exercise.name }}
                  </h4>
                </a>
              </div>

              <!-- Card body -->
              <div
                id="collapse30"
                class="collapse show"
                role="tabpanel"
                aria-labelledby="heading30"
                data-parent="#accordionEx5"
              >
                <div class="card-body grey lighten-3 black-text z-depth-1">
                  <div class="row justify-content-center">
                    <div class="col">
                      <div
                        class="card px-2 py-2"
                        style="background-color: transparent"
                      >
                        <div class="col">
                          DESCRIPTION:<br />
                          <p id="text">
                            <span v-html="exercise.description"> </span>
                          </p>
                        </div>
                      </div>
                    </div>
                  </div>
                  <div class="row mt-5">
                    <div class="col">
                      MUSCLES TRAINED:<br />
                      <div id="text" v-for="id in exercise.muscles" :key="id">
                        {{ muscles[id].name }}
                      </div>
                    </div>

                    <div class="col">
                      EQUIPMENT USED:<br />
                      <div id="text" v-for="id in exercise.equipment" :key="id">
                        {{ excal[id].name }}
                      </div>
                    </div>
                    <div class="col">
                      CALORIE BURNED:<br />
                      <div id="text">
                        {{ exercise.caloriesBurnt }}
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <!-- Accordion card -->
          </div>
          <!--/.Accordion wrapper-->
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";
import { mapGetters } from "vuex";
import firebase from "../../firebase.js";
import muscles from "../../assets/muscles.json";
import excal from "../../assets/excal.json";

export default {
  computed: {
    ...mapGetters({
      exRequest: "exercise",
      user: "user",
    }),
  },

  data() {
    return {
      exercises: [],
      equipmentList: [],
      intensity: "",
      muscles,
      excal,
      date: "",
      requeststring:
        "https://wger.de/api/v2/exercise/?language=2&format=json&limit=5&equipment=7",
      totalCaloriesBurnt: 0,
    };
  },

  methods: {
    async tracker(intensity, cal) {
      const userid = this.user.data.id;
      var update = {};
      update[`${this.date}.expend`] = this.totalCaloriesBurnt;
      const db = firebase.firestore().collection("calories").doc(userid);
      const db2 = await db.get();
      if (db2.exists) {
        db.update(update);
      } else {
        var update2 = {};
        update2[`${this.date}`] = {
          expend: this.totalCaloriesBurnt,
        };
        db.set(update2);
      }
      //console.log("HI");
      intensity = intensity[0].toUpperCase() + intensity.substring(1);
      alert(
        intensity +
          " intensity workout completed!\n" +
          cal.toString() +
          " kcal of calories will be logged."
      );
    },
    calculateCalories: function (exercise) {
      const id = exercise.equipment[0];
      const calorieMin = excal[id][this.intensity];
      const totalMin =
        this.intensity == "light"
          ? 25 / 5 //total duration minus rest time
          : this.intensity == "moderate"
          ? 38.75 / 5
          : 52.5 / 5;
      const caloriesBurnt =
        Math.round(
          (calorieMin * totalMin + Math.random() - Math.random() + 1) * 10
        ) / 10;
      return { ...exercise, caloriesBurnt };
    },
    requestStringMaker: function () {
      for (let equipment of this.equipmentList) {
        var id =
          equipment == "Pull-Up Bar"
            ? 6
            : equipment == "Dumbbell"
            ? 3
            : equipment == "Barbell"
            ? 1
            : 10;
        this.requeststring += "&equipment=" + id;
      }
      this.requeststring +=
        "&offset=" + Math.floor((Math.random() * 100 + 1) / 6);
      //console.log(this.requeststring);
      return this.requeststring;
    },
  },

  mounted() {
    const { intensity, equipments } = this.exRequest;
    this.equipmentList = equipments;
    this.intensity = intensity;
    axios.get(this.requestStringMaker()).then((response) => {
      const exercises = response.data.results;
      this.exercises = exercises.map(this.calculateCalories);
      const totalCaloriesBurnt = this.exercises.reduce(
        (accumulator, exercise) => accumulator + exercise.caloriesBurnt,
        0
      );
      this.totalCaloriesBurnt = Math.round(totalCaloriesBurnt * 100) / 100;
      //console.log(this.exercises);
      //console.log(this.totalCaloriesBurnt);
      // Curr Date Calculation
      var today = new Date();
      var dd = String(today.getDate()).padStart(2, "0");
      var mm = String(today.getMonth() + 1).padStart(2, "0"); //January is 0!
      var yyyy = today.getFullYear();
      today = yyyy + "-" + mm + "-" + dd;
      this.date = today;
    });
  },
};
</script>
<style scoped>
#title {
  font-family: Arial, Helvetica, sans-serif;
}

.accordion {
  font-family: "Caveat", cursive;
}
#text {
  font-family: Arial, Helvetica, sans-serif;
}

.btn {
  position: absolute;
  bottom: 25px;
  right: 15px;
}

p {
  font-size: 12pt;
  font-family: Arial;
}

.col-7 {
  font-size: 14pt;
}

.ing {
  margin-top: 0;
  margin-bottom: 0;
  font-family: Arial;
  font-size: 12pt;
}
.button {
  z-index: 1000;
}
</style>
