<template>
  <v-content>
    <v-container fluid fill-height>
      <v-layout align-center justify-center>
        <v-flex xs12 sm12 md8>

          <v-dialog
            v-model="authErrorDialog"
            max-width="500"
          >
            <v-card color="#EF9A9A">
              <v-card-title class="headline ">Authorization Error</v-card-title>

              <v-card-text>
                <div class="title">
                  {{ authErrorDialogText }}
                </div>
                <div class="body-1">
                  {{ authErrorDialogDescriptionText }}
                </div>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>

                <v-btn
                  color="white"
                  text
                  @click="authErrorDialog = false"
                >
                  Try Again
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>


          <v-stepper
                  v-model="currentStep"
                  :vertical="false"
                  :alt-labels="true"
          >

            <v-stepper-header>
              <template v-for="s in steps">
                <v-stepper-step
                        :key="`${s.key}-step`"
                        :complete="currentStep > s.key"
                        :step="s.key"
                        :editable="s.editable"
                        :rules="s.rules"
                >
                  {{ s.header }}
                </v-stepper-step>

                <v-divider
                        v-if="s !== steps"
                        :key="s.key"
                />
              </template>
            </v-stepper-header>

            <v-stepper-items>

              <!-- Step 1 - Collect farmOS Server URL -->
              <v-stepper-content
                      :key="`1-content`"
                      :step="1"
              >
                  <v-card>
                    <v-card-text>
                      <template>
                        Enter the URL of your farmOS server
                      </template>
                      <form data-vv-scope="urlForm">
                        <v-text-field
                              v-model="farmUrl"
                              v-validate="{ required: true, url: { require_protocol: true } }"
                              data-vv-name="farmUrl"
                              data-vv-scope="urlForm"
                              :error-messages="errors.first('urlForm.farmUrl')"
                              required
                              hint="https://yourfarm.farmos.net"
                        />
                      </form>
                    </v-card-text>

                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn @click="cancel">Cancel</v-btn>
                      <v-btn
                              color="primary"
                              @click="submitUrl"
                      >
                        Next
                      </v-btn>
                    </v-card-actions>
                  </v-card>
              </v-stepper-content>


              <!-- Step 2 - Authorize farmOS Server -->
              <v-stepper-content
                      :key="`2-content`"
                      :step="2"
              >
                <v-card>
                  <v-card-text>
                    <FarmAuthorizationForm
                            ref="authForm"
                            v-bind:authStarted.sync="authStarted"
                            v-bind:authFinished.sync="authFinished"
                            v-bind:appName="appName"
                            v-bind:redirectUri="'/register-farm'"
                            v-bind:apiToken.sync="apiToken"
                            v-bind:farmUrl.sync="farmUrl"
                            v-bind:farmName.sync="farmName"
                            v-bind:scope.sync="scope"
                            v-bind:authtoken.sync="authToken"
                            v-bind:farminfo.sync="farmInfo"
                            v-on:authorizationcomplete="currentStep = 3"
                    />
                  </v-card-text>
                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                            color="primary"
                            @click="$refs.authForm.openSignInWindow()"
                            :loading="authStarted === true && authFinished !== true"
                            :disabled="authStarted === true || authFinished === true"
                    >
                      Authorize
                    </v-btn>
                  </v-card-actions>

                </v-card>
              </v-stepper-content>


              <!-- Step 3 - Collect Farm Profile Info -->
              <v-stepper-content
                      :key="`3-content`"
                      :step="3"
              >
                <v-card>
                  <v-card-text>
                    <form data-vv-scope="farmInfoForm">
                      <v-text-field
                              label="Farm Name"
                              v-model="farmName"
                              v-validate="'required'"
                              data-vv-name="name"
                              data-vv-scope="farmInfoForm"
                              :error-messages="errors.first('farmInfoForm.name')"
                              required
                      />
                      <v-text-field
                              label="URL"
                              v-model="farmUrl"
                              v-validate="{ required: true, url: {require_protocol: true } }"
                              data-vv-name="url"
                              data-vv-scope="farmInfoForm"
                              :error-messages="errors.first('farmInfoForm.url')"
                              required
                              readonly
                      />
                      <v-text-field
                              label="API Version"
                              v-model="farmInfo.api_version"
                              data-vv-name="api_version"
                              data-vv-scope="farmInfoForm"
                              :error-messages="errors.first('farmInfoForm.api_version')"
                              required
                              readonly
                      />
                      <v-text-field
                              label="Tags (Optional)"
                              v-model="tags"
                              data-vv-name="tags"
                              data-vv-scope="farmInfoForm"
                              :error-messages="errors.first('farmInfoForm.tags')"
                      />
                      <div>
                        <FarmTagsChips v-bind:tags="tags"/>
                      </div>
                    </form>
                  </v-card-text>
                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                            color="primary"
                            @click="submitFarmInfo"
                    >
                      Next
                    </v-btn>
                  </v-card-actions>

                </v-card>
              </v-stepper-content>

              <!-- Step 4 - Verify and Save Farm Profile Info -->
              <v-stepper-content
                      :key="`4-content`"
                      :step="4"
              >
                <v-card>
                  <v-card-text>
                    <form data-vv-scope="farmVerifyForm">
                      <v-text-field
                              label="Farm Name"
                              v-model="farmName"
                              v-validate="'required'"
                              data-vv-name="name"
                              data-vv-scope="farmVerifyForm"
                              :error-messages="errors.first('farmVerifyForm.name')"
                              required
                      />
                      <v-text-field
                              label="URL"
                              v-model="farmUrl"
                              v-validate="{ required: true, url: {require_protocol: true } }"
                              data-vv-name="url"
                              data-vv-scope="farmVerifyForm"
                              :error-messages="errors.first('farmVerifyForm.url')"
                              required
                              readonly
                      />
                      <v-text-field
                              label="API Version"
                              v-model="farmInfo.api_version"
                              data-vv-name="api_version"
                              data-vv-scope="farmVerifyForm"
                              :error-messages="errors.first('farmVerifyForm.api_version')"
                              required
                              readonly
                      />
                      <v-text-field
                              label="Tags (Optional)"
                              v-model="tags"
                              data-vv-name="tags"
                              data-vv-scope="farmVerifyForm"
                              :error-messages="errors.first('farmVerifyForm.tags')"
                      />
                      <div>
                        <FarmTagsChips v-bind:tags="tags"/>
                      </div>
                    </form>
                  </v-card-text>
                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                            color="primary"
                            :disabled="farmCreated"
                            @click="createFarm"
                    >
                      Save
                    </v-btn>
                  </v-card-actions>

                </v-card>

                <!-- Dialog for presenting successful creation -->
                <v-dialog v-model="successDialog" persistent max-width="450">
                  <v-card>
                    <v-card-title class="headline">Farm Registered!</v-card-title>
                    <v-card-text>
                      <p>
                        Your farm, {{ farmName }}, was successfully registered with the {{ appName }}!
                      </p>

                      <p class="font-weight-black">
                        Your farm has ID={{ farmID }}
                      </p>

                      <p>
                        No further action is required. Administrators of the {{ appName }} must approve your farm before its data is used.
                      </p>

                    </v-card-text>
                  </v-card>
                </v-dialog>

              </v-stepper-content>


            </v-stepper-items>
          </v-stepper>

        </v-flex>
      </v-layout>
    </v-container>
  </v-content>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { FarmProfileCreate, FarmToken, FarmInfo } from '@/interfaces';
import { env } from '@/env';
import { commitAddNotification } from '@/store/main/mutations';
import {
    dispatchCreateFarm,
    dispatchValidateFarmUrl,
} from '@/store/farm/actions';
import FarmAuthorizationForm from '@/components/FarmAuthorizationForm.vue';
import FarmTagsChips from '@/components/FarmTagsChips.vue';

@Component({
  components: {FarmAuthorizationForm, FarmTagsChips},
})
export default class RegisterFarm extends Vue {
  public $refs!: {
      authForm: HTMLFormElement,
  };

  public appName = env('appName');
  public openFarmRegistration = env('openFarmRegistration');
  public inviteFarmRegistration = env('inviteFarmRegistration');
  // Query params.
  public apiToken: string = '';

  // Initialize variables for farm profile.
  public farmName: string = '';
  public farmUrl: string = '';
  public tags: string = '';
  public scope: string = '';

  // Initialize variables for the authorization step.
  public authStarted: boolean = false;
  public authFinished: boolean = false;
  public authCode: string = '';
  public authState: string = '';
  public authError: string = '';
  public authErrorDialog: boolean = false;
  public authErrorDialogText: string = '';
  public authErrorDialogDescriptionText: string = '';
  public authErrorDescription: string = '';
  public authToken: FarmToken = {
    access_token: '',
    refresh_token: '',
    expires_at: '',
    expires_in: '',
  };
  public farmInfo: FarmInfo = {};

  // Variables for success dialog
  public successDialog: boolean = false;
  public farmCreated: boolean = false;
  public farmID: number|null = null;

  public currentStep: number = 1;
  public steps: object[] = [
    {
      key: 1,
      header: 'farmOS URL',
      complete: false,
      editable: false,
      rules: [],
    },
    {
      key: 2,
      header: 'Authorize',
      complete: false,
      editable: false,
      rules: [],
    },
    {
      key: 3,
      header: 'Farm Info',
      complete: false,
      editable: false,
      rules: [],
    },
    {
      key: 4,
      header: 'Verify',
      complete: false,
      editable: false,
      rules: [],
    },

  ];

  public async mounted() {
    // Check for authorization errors.
    this.authError = this.$router.currentRoute.query.error as string;
    this.authErrorDescription = this.$router.currentRoute.query.error_description as string;
    if (this.authError) {
        if (this.authError === 'invalid_scope') {
            this.authErrorDialogText = 'OAuth Scope Error';
            this.authErrorDialogDescriptionText = 'Your farmOS server does not have an Authorization Scope enabled. Ensure this is enabled in /admin/config/farm/oauth ';
            this.authErrorDialog = true;

        } else if (this.authError === 'access_denied') {
            this.authErrorDialogText = 'Authorization Denied';
            this.authErrorDialogDescriptionText = 'You denied the Authorization request. In order to add your farm to the ' + this.appName + ', you must authorize access to your farmOS Server.';
            this.authErrorDialog = true;
        } else {
            commitAddNotification(this.$store, {
                content: this.authError + ': ' + this.authErrorDescription,
                color: 'error',
            });
        }
    }

    // Get authorization code and authorization state
    this.authCode = this.$router.currentRoute.query.code as string;
    this.authState = this.$router.currentRoute.query.state as string;
    if (this.authCode && this.authState) {
        this.currentStep = 2;
        // Finish authorization in the FarmAuthorizationFrom component.
        this.$refs.authForm.finishAuthorization(this.authCode, this.authState);
        return;
    }

    // Save an API Token if provided.
    this.apiToken = this.$router.currentRoute.query.api_token as string;

    // Check farm registration configuration.
    if (!this.openFarmRegistration && this.inviteFarmRegistration) {
        // Verify an APIToken was provided.
        if (!this.apiToken) {
            commitAddNotification(this.$store, {
                content: 'You cannot register farms without an invitation from the administrator.',
                color: 'error',
            });
            this.$router.push('/');
        }
    }

    if (!this.openFarmRegistration && !this.inviteFarmRegistration) {
        commitAddNotification(this.$store, {
            content: 'You cannot join this aggregator without an invitation from the administrator.',
            color: 'error',
        });
        this.$router.push('/');
        return;
    }

    // Save the farm url from query params.
    this.farmUrl = this.$router.currentRoute.query.farmUrl as string;

    // If a farmUrl is provided, assume that it is valid.
    // This is used to help automate the authorization process,
    // Such as when requesting authorization via email.
    if (this.farmUrl) {
        this.currentStep = 2;
    }


    this.reset();
  }

  public cancel() {
    this.$router.push('/');
  }

  public reset() {
    this.$validator.reset();
  }

  public async submitUrl() {
      this.$validator.validateAll('farmUrl').then( (isValid) => {
          if (isValid) {
              dispatchValidateFarmUrl(
                  this.$store,
                  {farmUrl: this.farmUrl, apiToken: this.apiToken },
              ).then( (response) => {
                  if (response) {
                      this.currentStep = 2;
                  }
              });
          }
      });
  }

  public submitFarmInfo() {
    this.$validator.validateAll('FarmInfoForm').then( (isValid) => {
      if (isValid) {
          this.currentStep ++;
      }
    });
  }

  public createFarm() {
    this.farmCreated = true;
    this.$validator.validateAll('FarmVerifyForm').then( (isValid) => {
        if (isValid) {
            const newFarm: FarmProfileCreate = {
                farm_name: this.farmName,
                url: this.farmUrl,
                tags: this.tags,
                token: this.authToken,
                scope: this.scope,
            };
            dispatchCreateFarm(this.$store, { data: newFarm, apiToken: this.apiToken } ).then(
                (success) => {
                    this.successDialog = true;
                    this.farmID = success.id;
                },
            );
        }
    });

  }
}
</script>
