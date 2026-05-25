<template>
  <Experiment title="Plural Cause Pretest 01">
    <InstructionScreen :title="'Welcome'">
      In this study, you will be presented with a fictional scenario and asked for your judgement of a sentence.
    </InstructionScreen>

    <InstructionScreen>
      <p>
        A new trade route is being planned between the planets Luteus and Viridis in a distant solar system.
      </p>

      <img src="images/planets.png"/>

      <p>
        This route can only officially be established with approval from the <b>Interplanetary Trade Council of Planets
        Luteus and Viridis</b>.
        Each planet has a representative and <b>both must approve</b> the route for it to be authorised.
      </p>

      <img src="images/council.png"/>
    </InstructionScreen>

    <template v-for="(trial, i) in comprehension_trials">
      <Screen :key="'comprehension-' + i">
        <Slide>
          <div v-if="($magpie.measurements['attempts_' + i] || 0) < 2">
            <span style='color:grey'>Remember that the route can only be established with the approval of the Council, meaning both members must agree.</span>

            <p>
              <img :src="trial.picture"/>
            </p>

            <p>
              Could the trade route be established given these results?
            </p>

            <ForcedChoiceInput
                :response.sync="$magpie.measurements['given_response_' + i]"
                :options="['Yes, it could.','No, it could not.']"
                @update:response="check_response($magpie.measurements['given_response_' + i], trial.correct_response, i)"/>

            <p v-if="$magpie.measurements['attempts_' + i] === 1" style="color: #B22222; font-weight: bold;">
              That is incorrect. Please consult the rule at the top of the screen and try again.
            </p>
          </div>

          <div v-else style="text-align: center; padding: 20px;">
            <p>You have answered the comprehension question incorrectly twice.</p>
            <p><b>Please close the study and click 'Stop Without Completing' on Prolific to return your submission.</b>
            </p>
          </div>

          <Record
              :data="{
              trial_type : 'comprehension',
              trial_num : i,
              scenario : trial.scenario,
              given_response : $magpie.measurements['given_response_' + i],
              correct_response: trial.correct_response,
              failed_comprehension: ($magpie.measurements['attempts_' + i] || 0) >= 2
            }"
          />
        </Slide>
      </Screen>
    </template>
    <InstructionScreen>
      <p>You have passed all comprehension questions! Now we can move on to the main part of the experiment.</p>
    </InstructionScreen>

    <Screen>
      <Slide>
        <p>
          The long-awaited day arrives as the Council delivers its
          decision on the proposed new trade route:
        </p>
        <img src="images/council_yes-yes.png"/>
        <p>An inhabitant of Glaucus, the third planet in this solar system, describes the result like this:
        </p>
        <img :src="granularity === 'coarse' ? 'images/glaucusian_coarse.png' : 'images/glaucusian_fine.png'"/>

        <Button v-if="!show_singular"
                @click="show_singular = true">
          Next
        </Button>
        <p v-if="show_singular">
          How likely is it that this individual would also agree with the following statement?
        </p>
        <p v-if="show_singular">
          <b>The route can be established because the representative from {{ singular_cause }} agreed to it.</b>
          <RatingInput
              left="Very unlikely"
              right="Very likely"
              :response.sync="$magpie.measurements.singular_response"
          />
        </p>

        <p v-if="$magpie.measurements.singular_response > 0">
          <button @click="$magpie.saveAndNextScreen();">Submit</button>
        </p>

        <Record
            :data="{
              trial_type : 'critical',
              granularity : granularity,
              singular_cause : singular_cause,
            }"
        />
      </Slide>
    </Screen>

    <PostTestScreen/>
    <SubmitResultsScreen/>
  </Experiment>
</template>

<script>
import Vue from 'vue';
import _ from 'lodash';
import comprehension_trials_ordered from './comprehension_trials.csv';

const granularity = _.sample(["coarse", "fine"]);
const singular_cause = _.sample(["Viridis", "Luteus"]);
const comprehension_trials = _.shuffle(comprehension_trials_ordered);

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
    _() {
      return _;
    }
  },
  methods: {
    check_response(given_response, correct_response, index) {
      if (this.$magpie.measurements['attempts_' + index] === undefined) {
        Vue.set(this.$magpie.measurements, 'attempts_' + index, 0);
      }
      if (given_response === correct_response) {
        this.$magpie.saveAndNextScreen();
      } else {
        this.$magpie.measurements['attempts_' + index] += 1;

        if (this.$magpie.measurements['attempts_' + index] < 2) {
          this.$magpie.measurements['given_response_' + index] = null;
        }
      }
    }
  }
};
</script>