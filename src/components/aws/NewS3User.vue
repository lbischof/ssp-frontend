<template>
    <div>
        <div class="hero is-light">
            <div class="hero-body">
                <div class="container">
                    <h1 class="title"><i class="material-icons">perm_identity</i> Create AWS S3 Bucket User</h1>
                </div>
                <h2 class="subtitle">
                    Here you can create a user for your AWS bucket. All orders will be logged and charged.</h2>
            </div>
        </div>
        <br>
        <form v-on:submit.prevent="newS3User">
            <b-field label="Name of the new user">
                <b-input type="text"
                         required pattern="^[a-zA-Z0-9\-]+$"
                         ref="autofocus"
                         v-model.number="username">
                </b-input>
            </b-field>
            <b-message type="is-info">
                The name of the new user will be put together with your details as follows:
                <br/>[BucketName]-[Benutzername]
                <br/><br/>Example: sbb-my-bucket-prod-user or sbb-my-app-prod-admin
            </b-message>

            <b-field label="Bucket Name">
                <b-select placeholder="Choose your bucket"
                          :loading="loading"
                          v-model="bucket"
                          required>
                    <option
                            v-for="bucket in buckets"
                            :value="bucket.name"
                            :key="bucket.name">
                        {{ bucket.name }}
                    </option>
                </b-select>
            </b-field>

            <label class="label">Permissions: read/write</label>
            <b-field>
                <b-radio-button v-model="isReadonly"
                                native-value="true"
                                type="is-success">
                    <span>Read-only</span>
                </b-radio-button>
                <b-radio-button v-model="isReadonly"
                                native-value="false"
                                type="is-danger">
                    <span>Read & write</span>
                </b-radio-button>
            </b-field>

            <button :disabled="loading"
                    v-bind:class="{'is-loading': loading}"
                    class="button is-primary">Create S3 user
            </button>
        </form>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                buckets: [],
                username: '',
                bucket: '',
                isReadonly: "true",
                loading: false
            };
        },
        mounted: function () {
            this.getUsersBuckets();
        },
        methods: {
            getUsersBuckets: function () {
                this.loading = true;
                this.$http.get(this.$store.state.backendURL + '/api/aws/s3').then((res) => {
                    this.buckets = res.body.buckets;
                    this.loading = false;
                }, () => {
                    this.loading = false;
                });
            },
            newS3User: function () {
                this.loading = true;

                this.$http.post(this.$store.state.backendURL + '/api/aws/s3/' + this.bucket + '/user', {
                    username: this.username,
                    isReadonly: this.isReadonly === 'true'
                }).then(() => {
                    this.loading = false;
                }, () => {
                    this.loading = false;
                });
            }
        }
    };
</script>
