<template>
    <div>
        <b-field>
            <template slot="label">
                Active Directory Group Name
                <b-tooltip type="is-dark" multilined animated position="is-right" :label="help">
                    <b-icon size="is-small" icon="help-circle-outline"></b-icon>
                </b-tooltip>
            </template>
            <b-autocomplete
                v-model="valueData"
                @input="inputChanged"
                :data="filteredDataArray"
                :loading="loading"
                :pattern="pattern"
                open-on-focus
                ref="el"
                required>
                <template slot="empty">No groups found</template>
            </b-autocomplete>
        </b-field>
        <!-- not clear why margin is missing -->
        <b-message style="margin-bottom: 1.5rem" type="is-info">
           <v-template v-if="isAdmin">
                You are in the group {{ admingroup }} and are therefore allowed to input any group (not only your own)<br>
           </v-template>
           <a href="https://confluence.sbb.ch/x/1pfxS">Learn how to manage your Active Directory groups</a>
        </b-message>
    </div>
</template>

<script>
  export default {
    name: 'ldap-groups',
    props: ['value', 'help', 'admingroup'],
    data() {
      return {
        valueData: this.value,
        groups: [],
        loading: false,
        pattern: ''
      };
    },
    mounted: function () {
        this.getGroups();
    },
    watch: {
        valueData: function(val) {
            // Fix for html5 validation not correctly working
            // when selecting from dropdown before entering any text
            // or selecting something, then removing characters and
            // selecting the same thing again.
            this.$nextTick(() => {
                this.$refs.el.$refs.input.checkHtml5Validity()
                this.$refs.el.checkHtml5Validity()
            })
        }
    },
    methods: {
      inputChanged: function(val) {
        if (this.isAdmin) {
            this.$emit('input', val)
        }
        if (this.groups.indexOf(val) > -1) {
            this.$emit('input', val)
        }
      },
      getGroups: function() {
        this.loading = true;
        this.$http.get(this.$store.state.backendURL + '/api/ldap/groups', null).then((res) => {
          this.groups = res.body.sort();
          if (this.isAdmin) {
            this.pattern = false
          } else {
            this.pattern = this.groups.join('|')
          }
          this.loading = false;
        }, () => {
          this.loading = false;
        });
      },
    },
    computed: {
      isAdmin() {
        if (this.groups.indexOf(this.admingroup) > -1) return true
        return false
      },
      filteredDataArray() {
        if (!this.groups) return
        return this.groups.filter((option) => {
          return option
            .toString()
            .toLowerCase()
            .indexOf(this.valueData.toLowerCase()) >= 0
        })
      }
    }
  };
</script>
