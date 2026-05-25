<template>
  <Experiment title="Plural Cause Pretest 01">
    <InstructionScreen :title="'Welcome'">
      In this experiment you will ask for your judgement of several statements based on a fictional scenario.
    </InstructionScreen>

    <InstructionScreen>
      <p>
        A new trade route is being planned between the planets Viridis and Luteus in a distant solar system.
      </p>

      <img src="images/planets.png"/>

      <p>
        This route can only officially be established with approval from the <b>Interplanetary Trade Council of Planets
        Luteus and Viridis</b>.
        Each planet has a delegate and <b>both must approve</b> the route for it to be authorised.
      </p>

      <img src="images/council.png"/>
    </InstructionScreen>


    <template v-for="(trial, i) in comprehension_trials">
      <Screen>
        <Slide>
          <span style='color:grey'>Remember that the route can only be established with Council approval meaning both members must agree.</span>


          <p>
            <img :src="trial.picture"/>
          </p>

          <p>
            Could the trade route be established given these results?
          </p>

          <ForcedChoiceInput
              :response.sync="$magpie.measurements.comp_response"
              :options="['Yes, it could.','No, it could not.']"
              @update:response="$magpie.saveAndNextScreen();"/>

          <Record
              :data="{
              trial_type : 'comprehension',
              trial_num : i+1,
              scenario : trial.scenario,
              given_response : $magpie.measurements.given_response,
              correct_response: trial.correct_response,
            }"
          />

        </Slide>
      </Screen>
    </template>


    <Screen>
      <Slide>
        <p>
          <p>The day when the new trade route gets decided on finally comes and this is the result:</p>
          <img src="images/council_yes-yes.png"/>
          <p>An inhabitant of a third planet in the solar system, Glaucus, describes the result like this:
          </p>
          <img :src="granularity === 'coarse' ? 'images/glaucusian_coarse.png' : 'images/glaucusian_fine.png'"/>

          <Button v-if="!show_singular"
                  @click="show_singular = true">
            Next
          </Button>
          <p v-if="show_singular">
            How likely is that inhabitant of Glaucus to agree with the following statement?
          </p>
          <p v-if="show_singular">
            <b>The route can be established because the representative from {{ singular_cause }} agreed.</b>
            <RatingInput
                left="XXX"
                right="YYY"
                :response.sync="$magpie.measurements.singular_response"
            />
          </p>

          <p v-if="$magpie.measurements.singular_response > 0">
            <button @click="$magpie.saveAndNextScreen();">Submit</button>
          </p>

          <Record
              :data="{
              trial_type : 'critical',
              trial_num : 1,
              granularity : granularity,
              singular_cause : singular_cause,
              // plural_response : $mapgie.measurements.plural_response,
              // singular_response : $magpie.measurements.singular_response
            }"
          />
      </Slide>
    </Screen>
    <!-- Speaker task template-->
    <!--    <Screen>-->
    <!--      <Slide>-->
    <!--        <p>-->
    <!--          The route is established because-->
    <!--          {{-->
    <!--            granularity === 'coarse' ? 'the Interplanetary Trade Council of planets Luteus and Viridis agreed.' : 'the representative of Luteus and the representative of Viridis agreed.'-->
    <!--          }}-->

    <!--          <div :style="{-->
    <!--          opacity: !show_singular ? 1 : 0.4,-->
    <!--          pointerEvents: !show_singular ? 'auto' : 'none'-->
    <!--          }">-->
    <!--            <RatingInput-->
    <!--                left="XXX"-->
    <!--                right="YYY"-->
    <!--                :response.sync="$magpie.measurements.plural_response"-->
    <!--            />-->
    <!--          </div>-->
    <!--        </p>-->
    <!--        <Button v-if="!show_singular"-->
    <!--                @click="show_singular = true">-->
    <!--          Next-->
    <!--        </Button>-->
    <!--        <p v-if="show_singular">-->

    <!--        </p>-->
    <!--        <p v-if="show_singular">-->
    <!--          The route can be established because the representative from {{ singular_cause }} agreed.-->
    <!--          <RatingInput-->
    <!--              left="XXX"-->
    <!--              right="YYY"-->
    <!--              :response.sync="$magpie.measurements.singular_response"-->
    <!--          />-->
    <!--        </p>-->

    <!--        <p v-if= "$magpie.measurements.singular_response > 0">-->
    <!--          <button @click="$magpie.saveAndNextScreen();">Submit</button>-->
    <!--        </p>-->

    <!--        <Record-->
    <!--            :data="{-->
    <!--              trial_type : 'critical',-->
    <!--              trial_num : 1,-->
    <!--              granularity : granularity,-->
    <!--              singular_cause : singular_cause,-->
    <!--              // plural_response : $mapgie.measurements.plural_response,-->
    <!--              // singular_response : $magpie.measurements.singular_response-->
    <!--            }"-->
    <!--        />-->
    <!--      </Slide>-->
    <!--    </Screen>-->

    <SubmitResultsScreen/>
  </Experiment>
</template>

<script>
import _ from 'lodash';

const granularity = _.sample(["coarse", "fine"])
console.log(granularity);

const singular_cause = _.sample(["Viridis", "Luteus"]);
console.log(singular_cause);

import comprehension_trials_ordered from './comprehension_trials.csv';

const comprehension_trials = _.shuffle(comprehension_trials_ordered);
console.log(JSON.parse(JSON.stringify(comprehension_trials_ordered)));

export default {
  name: 'App',
  data() {
    return {
      granularity: granularity,
      singular_cause: singular_cause,
      comprehension_trials: comprehension_trials,
      show_singular: false,

    };
  },
  computed: {
    // Expose lodash to template code
    _() {
      return _;
    }
  }
};
</script>
