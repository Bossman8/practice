<style>
#medicalTreatmentModal .popover {
  width: 50%;
  max-width: 80%;
}
#medicalTreatmentModal .modal-body {
  padding-left: 17px;
  padding-right: 17px;
  padding-top: 0;
  padding-bottom: 0;
}
.medical-treatment-documents .kt-widget4__item,
.medical-treatment-photos .kt-widget4__item {
  border-bottom: 0.07rem dashed #ebedf2 !important;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
}
.medical-treatment-documents .kt-widget4__item:hover,
.medical-treatment-photos .kt-widget4__item:hover {
  background-color: ghostwhite;
}
.medical-treatment-photos .kt-widget4__img img {
  border-radius: 5% !important;
  width: 3.5rem;
  height: 3.5rem;
}
#medicalTreatmentModal .tooltip-inner {
  text-align: left;
  padding: 2px;
  border: 0;
}
#medicalTreatmentModal .tooltip.show {
  opacity: 1;
}
#medicalTreatmentModal .tooltip-inner img {
  max-width: 150px;
  max-height: 150px;
}
.kt-widget4__img--icon > span.fa {
  font-size: 1.8em !important;
}
</style>
<style scoped>
.btn.btn-link:focus,
.btn.btn-link:hover,
.btn.btn-link.active {
  background-color: transparent;
}
.text-size-16 {
  font-size: 16px;
  font-weight: 500;
}
</style>

<template>
  <b-modal
    :no-enforce-focus="true"
    :no-close-on-esc="true"
    :no-close-on-backdrop="true"
    ref="modalRef"
    id="medicalTreatmentModal"
    hide-footer
    @hide="onHideModal"
    @show="onShowModal"
    @shown="onShownModal"
    size="lg"
    body-class
    :title="record && record.id? 'Edit Medical Treatment Record': 'Add New Medical Treatment'"
  >
    <form class="kt-form" @submit.prevent="saveRecord">
      <div class="kt-portlet__body">
        <div class="row kt-padding-10 kt-padding-b-0">
          <div class="col-12 d-flex">
            <label class="col-form-label kt-margin-r-10">Incident?</label>
            <div>
              <span
                class="kt-switch kt-switch--sm kt-switch--danger kt-switch--outline kt-switch--icon"
              >
                <label>
                  <input type="checkbox" v-model="record.incident" :disabled="record.id" />
                  <span></span>
                </label>
              </span>
            </div>
            <div v-if="record.incident && !record.id" class="kt-font-info kt-margin-l-5">
              Notice! Link to medical record will be emailed to
              <span
                class="kt-font-bolder"
              >medical-incident@cinnamonhills.org</span> when saved.
            </div>
          </div>
        </div>
        <div class="row kt-padding-10">
          <div class="col-md-6">
            <label>Student *</label>
            <multiselect
              v-model="record._student"
              :multiple="false"
              :custom-label="transformStudent"
              track-by="id"
              :options="students || []"
              :show-labels="false"
              :allow-empty="false"
              :disabled="loadingStudents || !!student"
              placeholder="Select Student"
            >
              <template slot="beforeList" slot-scope="props" v-if="!students || !students.length">
                <span class="text-muted kt-padding-l-5">No Records Available!</span>
              </template>
            </multiselect>
          </div>
          <div class="col-md-6">
            <label>Staff Treating (Default: You)</label>
            <multiselect
              v-model="record._staff_treating"
              :multiple="false"
              :custom-label="transformStaff"
              track-by="id"
              :options="staffs || []"
              :show-labels="false"
              :allow-empty="true"
              :disabled="loadingStaffs"
              placeholder="Staff Treating(You as default)"
            >
              <template slot="clear" slot-scope="props">
                <span
                  class="la la-close multiselect__clear"
                  title="Clear"
                  v-if="record._staff_treating"
                  @click.stop="record._staff_treating=null"
                ></span>
              </template>
              <template slot="beforeList" slot-scope="props" v-if="!staffs || !staffs.length">
                <span class="text-muted kt-padding-l-5">No Records Available!</span>
              </template>
            </multiselect>
          </div>
        </div>
       <div class="row kt-padding-10" v-if="!record.id">
          <div class="col-6">
            <label>Treatment Type:</label>

              <multiselect
              :disabled="record.id"
              v-model="treatment"
              :multiple="false"
              :options="treatment_options"
              label="title"
              >
              </multiselect>

          </div>
          <div class="col-6">
            <label>Subjective, Chief complaint(*):</label>
            <input
              type="text"
              class="form-control"
              placeholder="Subjective, Chief complaint"
              required
              v-model="record.subjective"
            />
          </div>
        </div>
        <div class="row kt-padding-10" v-else>
          <div class="col-12">
            <label>Subjective, Chief complaint(*):</label>
            <input
              type="text"
              class="form-control"
              placeholder="Subjective, Chief complaint"
              required
              v-model="record.subjective"
            />
          </div>
        </div>
        <div class="row kt-padding-10">
          <div class="col-12">
            <label>Note(*):</label>
            <!--<vue-summernote class="form-control" placeholder="Note" required v-model="record.note" :config="{dialogsInBody: false, height: '150'}"></vue-summernote>-->
            <vue-ckeditor2 v-model="record.note" :config="ckeditorConfig" />
          </div>
        </div>
        <div class="row kt-padding-10">
          <div class="col-md-6">
            <label>Date/Time(*):</label>
            <div class="input-group date">
              <vue-datetimepicker
                v-model="record.datetime"
                placeholder="Date/Time"
                :wrap="true"
                :config="{format: 'YYYY-MM-DD HH:mm', formatTime:'h:mm A', mask: true, step: 5, iconRef: '.datepicker-toggle'}"
              ></vue-datetimepicker>
              <div class="input-group-append datepicker-toggle">
                <span class="input-group-text">
                  <i class="la la-calendar-o glyphicon-th"></i>
                </span>
              </div>
            </div>
          </div>
          <div class="col-md-6">
            <label>Specialty(*):</label>
            <select class="form-control" v-model="record.specialty" required>
              <option v-for="e in specialtyOptions" :value="e.value">{{ e.title }}</option>
            </select>
          </div>
        </div>
        <div class="kt-separator kt-separator--space-sm kt-separator--border"></div>
        <div class="kt-padding-10">
          <h3 class="kt-form__section text-danger">
            <b-btn
              v-b-toggle.medicalTreatmentVitalRecordContainer
              variant="link"
              class="text-size-16 nounderline text-danger kt-form__section kt-padding-0"
            >
              <i class="fa fa-thermometer-half"></i>
              Vital Medical Record
              <i class="la la-angle-up when-opened"></i>
              <i class="la la-angle-down when-closed"></i>
              <sub class="kt-font-metal">Click to expand/collapse</sub>
            </b-btn>
          </h3>
        </div>
        <b-collapse :visible="false" id="medicalTreatmentVitalRecordContainer">
          <div class="row kt-padding-10">
            <div class="col-md-6">
              <label>Weight(lbs)</label>
              <div class="input-group">
                <input
                  type="number"
                  min="45"
                  max="450"
                  class="form-control"
                  placeholder="Weight(lbs)"
                  v-model="record.vital_record.weight"
                />
                <div class="input-group-append">
                  <span class="input-group-text">lbs</span>
                </div>
              </div>
            </div>
            <div class="col-md-6">
              <label>Height(ft/in)</label>
              <vue-feet-inch-input :feet-min="2" v-model="record.vital_record.height"></vue-feet-inch-input>
            </div>
          </div>
          <div class="row kt-padding-10">
            <div class="col-sm-6">
              <label>Temperature(°F)</label>
              <div class="input-group">
                <input
                  type="number"
                  min="75"
                  max="125"
                  class="form-control"
                  placeholder="Temperature(°F)"
                  v-model="record.vital_record.temperature"
                />
                <div class="input-group-append">
                  <span class="input-group-text">°F</span>
                </div>
              </div>
            </div>
            <div class="col-sm-6">
              <label>Blood Pressure upper/lower(mmHg)</label>
              <div class="input-group">
                <input
                  type="number"
                  min="20"
                  max="200"
                  class="form-control"
                  placeholder="Upper(mmHg)"
                  v-model="record.vital_record.blood_pressure_upper"
                />
                <div class="input-group-append">
                  <span class="input-group-text">mmHg</span>
                </div>
                <input
                  type="number"
                  min="20"
                  max="200"
                  class="form-control"
                  placeholder="Lower(mmHg)"
                  v-model="record.vital_record.blood_pressure_lower"
                />
                <div class="input-group-append">
                  <span class="input-group-text">mmHg</span>
                </div>
              </div>
            </div>
          </div>
        </b-collapse>
        <div class="kt-separator kt-separator--space-sm kt-separator--border"></div>
        <div class="kt-padding-10">
          <h3 class="text-size-16 kt-form__section text-danger">
            <i class="la la-check-circle"></i> Sign Record
          </h3>
        </div>
        <div class="row kt-padding-10">
          <div class="col-12">
            <label>
              I
              <span class="text-danger">"{{ record.signed_by || '__________' }}"</span> Sign this Medical record on
              <span
                class="text-danger"
              >"{{ formatDate(record.signed_at || nowDT({asMoment: true}), 'MMM D YYYY hh:mm A') }}"</span>.
            </label>
            <input
              type="text"
              placeholder="Enter Your Name"
              class="form-control"
              v-model="record.signed_by"
            />
          </div>
        </div>
        <div class="kt-separator kt-separator--space-sm kt-separator--border"></div>
        <div class="kt-padding-10">
          <h3 class="text-size-16 kt-form__section text-danger">
            <i class="la la-file-photo-o"></i> Photos
          </h3>
        </div>
        <div class="row kt-padding-10" v-if="allowedEdit">
          <div class="col-md-12">
            <label style="display: flex;">Add New Photo</label>
            <input
              type="text"
              class="form-control attachment-title"
              v-model="record.photo_attachment_title"
              placeholder="Title (Optional)"
            />
            <label class="btn btn-default btn-sm kt-margin-l-5 kt-margin-b-0" :disabled="saving">
              <span class="la la-file-photo-o"></span>
              Choose Photo(s)
              <input
                type="file"
                ref="photoFileRef"
                name="photo"
                accept="image/jpeg, image/png"
                style="display: none;"
                @change="onChangePhoto"
                multiple
              />
            </label>
            <span v-if="!!chosenPhotos.length" class="kt-margin-l-5">
              {{ chosenPhotos.length == 1? chosenPhotos[0].name: `${chosenPhotos.length} photos selected`}}
              <button
                type="button"
                class="btn btn-outline-hover-info"
                @click="clearChosenPhoto"
                title="Clear chosen image"
              >
                <span class="la la-close"></span>
              </button>
            </span>
            <span v-else class="text-muted kt-margin-t-5 kt-margin-l-5">No Photo Chosen</span>
          </div>
        </div>
        <div
          class="row medical-treatment-photos kt-padding-10"
          v-if="record.id && record.photo_attachments"
        >
          <div class="col-md-12">
            <div class="kt-widget4" v-for="photo in record.photo_attachments" :key="photo.id">
              <div class="kt-widget4__item">
                <div class="kt-widget4__img kt-widget4__img--logo">
                  <img
                    :src="photo.photo_thumb"
                    v-b-tooltip:medicalTreatmentModal.right.html.hover
                    :title="`<img src='${photo.photo_thumb}'>`"
                  />
                </div>
                <div class="kt-widget4__info">
                  <span class="kt-widget4__text">
                    {{ photo.title || '[NO TITLE]' }}
                    <br />
                    <small>{{ photo.file_name }}</small>
                  </span>
                </div>
                <div class="kt-widget4__ext" v-if="allowedEdit">
                  <span
                    ref="deleteDocumentConfirm"
                    @confirmed="deletePhoto(photo)"
                    is="inline-confirm"
                    btn-class="kt-widget4__icon kt-portlet__nav-link btn kt-btn kt-btn--hover-danger kt-btn--icon kt-btn--icon-only kt-btn--pill"
                    sure-class="text-danger kt-margin-r-5 kt-margin-t-5"
                    yes-class="btn-danger btn-sm"
                    no-class="btn-default btn-sm"
                    confirming-icon-class="la la-spin la-spinner"
                    class="d-inline-flex"
                    :confirming="deletingDocument"
                    :disabled="$cannot('delete', 'medicaltreatment')"
                  >
                    <i class="la la-trash text-danger"></i>
                  </span>
                </div>
                <div class="kt-widget4__ext">
                  <a
                    :href="photo.photo"
                    target="_blank"
                    class="kt-widget4__icon kt-portlet__nav-link btn kt-btn kt-btn--hover-info kt-btn--icon kt-btn--icon-only kt-btn--pill"
                    title="Download Photo"
                  >
                    <i class="la la-download text-info"></i>
                  </a>
                </div>
              </div>
            </div>
            <div v-if="!record.photo_attachments.length" class="text-center">
              <span class="text-muted font-italic">No Photo!</span>
            </div>
          </div>
        </div>
        <div class="kt-separator kt-separator--space-sm kt-separator--border"></div>
        <div class="kt-padding-10">
          <h3 class="text-size-16 kt-form__section text-danger">
            <i class="la la-file-o"></i> Document Attachments
          </h3>
        </div>
        <div class="row kt-padding-10" v-if="allowedEdit">
          <div class="col-md-12">
            <label style="display: flex;">Attach New Documents</label>
            <input
              type="text"
              class="form-control attachment-title"
              v-model="record.attachment_title"
              placeholder="Title (Optional)"
            />
            <label class="btn btn-default btn-sm kt-margin-l-5 kt-margin-b-0" :disabled="saving">
              <span class="la la-file-o"></span>
              Choose File(s)
              <input
                type="file"
                ref="documentFileRef"
                name="file"
                style="display: none;"
                @change="onChangeFile"
                multiple
              />
            </label>
            <span v-if="!!chosenFiles.length" class="kt-margin-l-5">
              {{ chosenFiles.length == 1? chosenFiles[0].name: `${chosenFiles.length} files selected`}}
              <button
                type="button"
                class="btn btn-outline-hover-info"
                @click="clearChosenFile"
                title="Clear chosen image"
              >
                <span class="la la-close"></span>
              </button>
            </span>
            <span v-else class="text-muted kt-margin-t-5 kt-margin-l-5">No File Chosen</span>
          </div>
        </div>
        <div
          class="row medical-treatment-documents kt-padding-10"
          v-if="record.id && record.attachments"
        >
          <div class="col-md-12">
            <div class="kt-widget4" v-for="attachment in record.attachments" :key="attachment.id">
              <div class="kt-widget4__item">
                <div
                  class="kt-widget4__img kt-widget4__img--icon"
                  v-html="formatFileType(attachment.file_name, false)"
                ></div>
                <div class="kt-widget4__info">
                  <span class="kt-widget4__text">
                    {{ attachment.title || '[NO TITLE]' }}
                    <br />
                    <small>{{ attachment.file_name }}</small>
                  </span>
                </div>
                <div class="kt-widget4__ext" v-if="allowedEdit">
                  <span
                    ref="deleteDocumentConfirm"
                    @confirmed="deleteDocument(attachment)"
                    is="inline-confirm"
                    btn-class="kt-widget4__icon kt-portlet__nav-link btn kt-btn kt-btn--hover-danger kt-btn--icon kt-btn--icon-only kt-btn--pill"
                    sure-class="text-danger kt-margin-r-5 kt-margin-t-5"
                    yes-class="btn-danger btn-sm"
                    no-class="btn-default btn-sm"
                    confirming-icon-class="la la-spin la-spinner"
                    class="d-inline-flex"
                    :confirming="deletingDocument"
                    :disabled="$cannot('delete', 'medicaltreatment')"
                  >
                    <i class="la la-trash text-danger"></i>
                  </span>
                </div>
                <div class="kt-widget4__ext">
                  <a
                    :href="attachment.file"
                    target="_blank"
                    class="kt-widget4__icon kt-portlet__nav-link btn kt-btn kt-btn--hover-info kt-btn--icon kt-btn--icon-only kt-btn--pill"
                    title="Download Document"
                  >
                    <i class="la la-download text-info"></i>
                  </a>
                </div>
              </div>
            </div>
            <div v-if="!record.attachments.length" class="text-center">
              <span class="text-muted font-italic">No Document!</span>
            </div>
          </div>
        </div>
      </div>
      <div class="modal-footer">
        <span
          ref="deleteConfirm"
          v-if="record.id && allowedEdit"
          @confirmed="deleteRecord"
          is="inline-confirm"
          btn-class="btn btn-danger mr-auto"
          sure-class="text-danger"
          yes-class="btn-danger"
          no-class="btn-default"
          confirming-icon-class="la la-spin la-spinner"
          class="inline mr-auto"
          :confirming="deleting"
          :disabled="$cannot('delete', 'medicaltreatment')"
        >
          <span>
            <i class="la la-trash"></i> Delete
          </span>
        </span>
        <button type="button" class="btn btn-secondary" @click="hide">
          <i class="la la-close"></i> Close
        </button>
        <template v-if="allowedEdit">
          <button
            v-if="record.id"
            type="submit"
            class="btn btn-warning"
            :disabled="saving || $cannot('change', 'medicaltreatment')"
          >
            <i :class="saving? 'la la-spin la-spinner':'la la-edit'"></i>
            <span v-show="!saving">Update</span>
            <span v-show="saving">Updating</span>
          </button>
          <button v-else type="submit" class="btn btn-success" :disabled="saving">
            <i :class="saving? 'la la-spin la-spinner':'la la-plus'"></i>
            <span v-show="!saving">Add</span>
            <span v-show="saving">Adding</span>
          </button>
        </template>
      </div>
    </form>
  </b-modal>
</template>

<script>
import $ from "jquery";
import Multiselect from "vue-multiselect";
import UtilMixin from "../../mixins/UtilMixin";
import RoutableModalMixin from "../../mixins/RoutableModalMixin";
import InlineConfirm from "../../libs/InlineConfirm";
import VueDatetimepicker from "../../libs/VueDatetimepicker";
import VueFeetInchInput from "../../libs/VueFeetInchInput";
import {DEFAULT_CKEDITOR_TOOLBAR, MEDICAL_TREATMENT_SPECIALTY_OPTIONS, MEDICAL_TREATMENT_TYPE} from "../../../Constants";
import VueCkeditor2 from "../../libs/VueCkeditor2";

const moment = window.moment;

const AUTO_FILLED_NOTE = `
<p><strong>Subjective continue:</strong></p>
<p></p>
<p><strong>Objective: </strong></p>
<p></p>
<p><strong>Assesment/Diagnosis:</strong></p>
<p></p>
<p><strong>Plan:</strong></p>
<p></p>
`;
export default {
  mixins: [UtilMixin, RoutableModalMixin],
  components: {
    InlineConfirm,
    VueDatetimepicker,
    VueCkeditor2,
    Multiselect,
    VueFeetInchInput
  },
  props: {
    student: {
      type: Object
    },
    allowedEdit: {
      type: Boolean,
      default: true
    }
  },
  data: function() {
    return {
      saving: false,
      deleting: false,
      loadingStudents: false,
      loadingStaffs: false,
      record: { _student: {}, vital_record: {} },
      deletingDocument: false,
      deletingPhoto: false,
      students: null,
      staffs: null,
      chosenFiles: [],
      chosenPhotos: [],
      loadingContacts: false,
      treatment: null,
      treatment_options: MEDICAL_TREATMENT_TYPE,
      specialtyOptions: MEDICAL_TREATMENT_SPECIALTY_OPTIONS,
      ckeditorConfig: {
        toolbar: DEFAULT_CKEDITOR_TOOLBAR,
        height: 200
      }
    };
  },
  watch: {
    treatment(val){
      if(val != null){
      this.record.note = val.value
      this.record.treatment_type = val.title
      }
      else{
        this.record.note = AUTO_FILLED_NOTE;
      }
      
    }
  },
  methods: {
    clearChosenFile: function() {
      this.chosenFiles = [];
      $(this.$refs.documentFileRef).val("");
    },
    clearChosenPhoto: function() {
      this.chosenPhotos = [];
      $(this.$refs.photoFileRef).val("");
    },
    onChangeFile: function(event) {
      if (event.target.files.length > 0) {
        this.chosenFiles = event.target.files || [];
      }
    },
    onChangePhoto: function(event) {
      if (event.target.files.length > 0) {
        this.chosenPhotos = event.target.files || [];
      }
    },
    transformStudent: function(student) {
      if (!student || !student.id) {
        return null;
      }
      return `${student.first_name} ${student.last_name}`;
    },
    transformStaff: function(staff) {
      if (!staff || !staff.id) {
        return null;
      }
      return `${staff.first_name} ${staff.last_name}`;
    },
    loadStaffs: function() {
      if (this.$cannot("view", "staff")) {
        return;
      }
      var self = this;
      self.loadingStaffs = true;
      this.$http
        .get("staff", {
          params: {
            page_size: 0,
            fields: "id,first_name,last_name",
            order_by: "first_name,last_name"
          }
        })
        .then(
          function(response) {
            self.staffs = response.data.results;
            self.loadingStaffs = false;
          },
          function(error) {
            self.showDefaultServerError(error);
            self.loadingStaffs = false;
          }
        );
    },
    loadStudents: function() {
      if (this.$cannot("view", "student") || this.student) {
        return;
      }
      var self = this;
      self.loadingStudents = true;
      this.$http
        .get("student", {
          params: {
            page_size: 0,
            fields: "id,first_name,last_name",
            status_exclude: ["denied", "discharged", "withdrawn_application"],
            order_by: "first_name,last_name"
          }
        })
        .then(
          function(response) {
            self.students = response.data.results;
            self.loadingStudents = false;
          },
          function(error) {
            self.showDefaultServerError(error);
            self.loadingStudents = false;
          }
        );
    },
    show: function(record) {
      this.record = Object.assign(
        {
          _student: this.student || {},
          vital_record: {},
          note: AUTO_FILLED_NOTE,
          datetime: new Date()
        },
        record || {}
      );
      if (this.record.id) {
        this.$set(
          this.record,
          "note",
          '<p><span style="color:#95a5a6">Loading ...</span></p>'
        );
      }
      this.treatment = null;
      this.chosenFiles = [];
      this.chosenPhotos = [];
      this.$refs.modalRef.show();
    },
    hide: function() {
      this.$refs.modalRef.hide();
    },
    loadRecord: function() {
      var self = this;
      this.$http.get(`medical_treatment/${this.record.id}`).then(
        function(response) {
          self.record = response.data;
        },
        function(error) {
          self.$set(
            self.record,
            "note",
            '<p><span style="color:#95a5a6">FAILED TO LOAD!</span></p>'
          );
          self.showDefaultServerError(error);
        }
      );
    },
    onHideModal: function() {
      this.saving = false;
      this.deleting = false;
      this.record = { _student: {}, vital_record: {} };
      this.routableModalSetRoute();
    },
    onShowModal: function() {
      this.saving = false;
      this.deleting = false;
      if (this.$refs.deleteConfirm) {
        this.$refs.deleteConfirm.confirm = false;
      }
      if (!this.staffs) {
        this.loadStaffs();
      }
      if (!this.students) {
        this.loadStudents();
      }
      this.routableModalSetRoute();
    },
    onShownModal: function() {
      if (this.record.id) {
        this.$nextTick(function() {
          this.loadRecord();
        });
      }
    },
    saveRecord: function() {
      var self = this,
        recordId = this.record.id;
      self.saving = true;
      var postData = Object.assign({}, this.record);
      postData["student"] = (this.record._student || {}).id;
      postData["staff_treating"] = (this.record._staff_treating || {}).id;
      if (postData.signed_by && !postData.signed_at) {
        postData.signed_at = this.nowDT({ asUtc: true });
      }
      postData.files = this.chosenFiles;
      postData.photos = this.chosenPhotos;
      var vitalRecord = {};
      [
        "id",
        "temperature",
        "blood_pressure_upper",
        "blood_pressure_lower",
        "weight",
        "height"
      ].forEach(function(f) {
        if (postData.vital_record[f]) {
          vitalRecord[f] = postData.vital_record[f];
        } else if (postData.vital_record["id"]) {
          vitalRecord[f] = "";
        }
      });
      postData.vital_record = vitalRecord;
      var formData = this.convertModelToFormData(
          postData,
          undefined,
          undefined,
          k =>
            k.startsWith("_") ||
            k === "attachments" ||
            k === "photo_attachments"
        ),
        headers = { headers: { "Content-Type": "multipart/form-data" } },
        url = "medical_treatment",
        httpFunc = this.$http.post,
        successMessage = "Medical Treatment record added successfully";
      if (recordId) {
        url = `medical_treatment/${recordId}/`;
        httpFunc = this.$http.patch;
        successMessage = "Medical Treatment record updated successfully";
      }
      httpFunc(url, formData, headers).then(
        function(response) {
          self.saving = false;
          self.showSuccess(successMessage, 5000);
          self.$emit("saved-success");
          self.hide();
        },
        function(error) {
          self.saving = false;
          self.showDefaultServerError(error);
        }
      );
    },
    deleteDocument: function(attachment) {
      var self = this,
        recordId = this.record.id;
      self.deletingDocument = true;
      this.$http
        .delete(`medical_treatment/${recordId}/attachment/${attachment.id}`)
        .then(
          function(response) {
            self.showSuccess("Document deleted successfully", 5000);
            self.deletingDocument = false;
            self.record.attachments = self.record.attachments.filter(
              a => a.id !== attachment.id
            );
          },
          function(error) {
            self.deletingDocument = false;
            self.showDefaultServerError(error);
          }
        );
    },
    deletePhoto: function(photo) {
      var self = this,
        recordId = this.record.id;
      self.deletingPhoto = true;
      this.$http
        .delete(`medical_treatment/${recordId}/photo_attachment/${photo.id}`)
        .then(
          function(response) {
            self.showSuccess("Photo deleted successfully", 5000);
            self.deletingPhoto = false;
            self.record.photo_attachments = self.record.photo_attachments.filter(
              a => a.id !== photo.id
            );
          },
          function(error) {
            self.deletingPhoto = false;
            self.showDefaultServerError(error);
          }
        );
    },
    deleteRecord: function() {
      var self = this;
      self.deleting = true;
      this.$http.delete(`medical_treatment/${self.record.id}`).then(
        function(response) {
          self.showSuccess(
            "Medical Treatment record deleted successfully",
            5000
          );
          self.hide();
          self.deleting = false;
          self.$emit("deleted-success");
        },
        function(error) {
          self.deleting = false;
          self.showDefaultServerError(error);
        }
      );
    }
  }
};
</script>
