<template>
  <div>
    <div class="container-fluid">
      <h1 class="h3 mb-2 text-gray-800">Quản Lý Môn Học</h1>
      <p class="mb-4 mt-4"></p>
      <div class="card shadow mb-4">
        <div class="card-header py-3">
          <h6 class="m-0 font-weight-bold text-primary">
            <el-button
              type="success"
              icon="fas fa-plus"
              circle
              @click="btnCreate"
              v-b-modal.modal-create
            ></el-button>
          </h6>
        </div>
        <div class="card-body">
          <div class="table-responsive">
            <el-table
              v-loading="loadingData"
              element-loading-text="Loading..."
              element-loading-spinner="el-icon-loading"
              element-loading-background="rgba(255, 255, 255, 0.7)"
              :data="listPagination"
            >
              <el-table-column label="SubjectID" prop="SubjectID"></el-table-column>
              <el-table-column label="Name Subject" prop="Name"></el-table-column>
              <el-table-column align="right">
                <template slot="header" slot-scope="scope">
                  <el-input v-model="search" size="mini" placeholder=" search" @input="searchName" />
                </template>
                <template slot-scope="scope">
                  <el-tooltip class="item" effect="dark" content="Detail" placement="top">
                    <router-link
                      :to="`subjects/${scope.row.SubjectID}/students`"
                      tag="button"
                      class="el-button el-tooltip item el-button--default el-button--mini is-round"
                    >
                      <i class="el-icon-info"></i>
                    </router-link>
                  </el-tooltip>
                  <el-tooltip class="item" effect="dark" content="Edit" placement="top">
                    <el-button
                      type="primary"
                      icon="el-icon-edit"
                      round
                      size="mini"
                      @click="info(scope.row, $event.target)"
                    ></el-button>
                  </el-tooltip>
                  <el-tooltip class="item" effect="dark" content="Delete" placement="top">
                    <el-button
                      size="mini"
                      type="danger"
                      icon="el-icon-delete"
                      round
                      @click="delSubject(scope.row)"
                    ></el-button>
                  </el-tooltip>
                </template>
              </el-table-column>
            </el-table>
          </div>

          <b-row>
            <div class="col-12 my-1 mt-3 pr-2 pl-2">
              <el-pagination
                :class="`float-md-right`"
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page.sync="currentPage"
                :page-sizes="pageOptions"
                :page-size="pageSize"
                layout="sizes, jumper, prev, pager, next"
                :total="total"
                small
              ></el-pagination>
            </div>
          </b-row>
        </div>
      </div>
    </div>

    <ValidationObserver ref="observer" v-slot="{ passes }">
      <b-modal
        :id="infoModal.id"
        ref="modal-edit"
        ok-title="Update"
        @ok.prevent="passes(handleUpdate)"
        title="Cập Nhật Môn Học"
      >
        <b-form>
          <ValidationProvider rules="required" name="Subject Name" v-slot="{ valid, errors }">
            <b-form-group>
              <b-form-input
                type="text"
                v-model="editSubject.Name"
                :state="errors[0] ? false : (valid ? true : null)"
                placeholder="Subject Name"
              ></b-form-input>
              <b-form-invalid-feedback id="inputLiveFeedback">{{ errors[0] }}</b-form-invalid-feedback>
            </b-form-group>
          </ValidationProvider>
        </b-form>
      </b-modal>
    </ValidationObserver>

    <ValidationObserver ref="observer" v-slot="{ passes }">
      <b-modal
        id="modal-create"
        ref="modal-create"
        title="Tạo Mới Môn Học"
        @ok.prevent="passes(handleCreate)"
        @cancel="resetInfoModalCreate"
        v-loading.fullscreen.lock="fullscreenLoading"
      >
        <b-form>
          <ValidationProvider rules="required" name="Subject Name" v-slot="{ valid, errors }">
            <b-form-group>
              <b-form-input
                type="text"
                v-model="newSubject.Name"
                :state="errors[0] ? false : (valid ? true : null)"
                placeholder="Subject Name"
              ></b-form-input>
              <b-form-invalid-feedback id="inputLiveFeedback">{{ errors[0] }}</b-form-invalid-feedback>
            </b-form-group>
          </ValidationProvider>
        </b-form>
      </b-modal>
    </ValidationObserver>
  </div>
</template>

<script>
import { mapState, mapActions } from "vuex";
import { ValidationObserver, ValidationProvider } from "vee-validate";
export default {
  components: {
    ValidationObserver,
    ValidationProvider
  },
  data() {
    return {
      editSubject: {
        Name: ""
      },
      newSubject: {
        Name: ""
      },
      infoModal: {
        id: "info-modal"
      },
      currentPage: 1,
      pageOptions: [10, 20, 50, 100],
      fullscreenLoading: false,
      pageSize: 10,
      search: "",
      loadingData: true,
      listQuery: [],
      listPagination:[],
      total:0
    };
  },
  methods: {
    ...mapActions("adminAcademy", [
      "getAllSubjects",
      "createSubject",
      "updateSubject",
      "deleteSubject"
    ]),
    handleSizeChange(val) {
      this.pageSize = val;
      this.setlistPagination();
    },
    handleCurrentChange(val) {
      this.currentPage = val ? val : 1;
      this.setlistPagination();
    },
    setlistPagination() {
      let startRecord = (this.currentPage - 1) * this.pageSize
      let endRecord = startRecord + this.pageSize;
      this.listPagination = this.listQuery.filter(
        (data , index ) =>
          index >= startRecord && index <= endRecord
      )
    },
    searchName() {
      this.listQuery = this.listSubjects.filter(
        data =>
          !this.search ||
          data.Name.toLowerCase().includes(this.search.toLowerCase())
      );
      this.setlistPagination();
    },
    info(item, button) {
      this.editSubject.Name = item.Name;
      this.$root.$emit("bv::show::modal", this.infoModal.id, button);
    },
    async handleCreate() {
      this.fullscreenLoading = true;
      let response = await this.createSubject(this.newSubject);
      if (response) {
        this.$refs["modal-create"].hide();
        await this.resetInfoModalCreate();
      }
      this.fullscreenLoading = false;
    },
    async handleUpdate() {
      this.$refs["modal-edit"].hide();
      await this.updateSubject(this.editSubject);
      await this.resetInfoModalEdit();
    },
    resetInfoModalEdit() {
      this.editSubject.Name = "";
    },
    resetInfoModalCreate() {
      this.newSubject.Name = "";
      requestAnimationFrame(() => {
        this.$refs.observer.reset();
      });
    },
    delSubject(subject) {
      this.$swal({
        title: "Are you sure?",
        text: "You won't be able to revert this!",
        type: "warning",
        showCancelButton: true,
        cancelButtonColor: "#d33",
        confirmButtonColor: "#28a745",
        confirmButtonText: "Yes, delete it!",
        reverseButtons: true
      }).then(result => {
        if (result.value) {
          this.deleteSubject(subject);
          this.$swal("Deleted!", "Your file has been deleted.", "success");
        }
      });
    },
    btnCreate(item, button) {
      this.$root.$emit("bv::show::modal", button);
    }
  },
  computed: {
    ...mapState("adminAcademy", ["listSubjects"])
  },
  async created() {
    let response = await this.getAllSubjects();
    this.total = this.listsubjects ? this.listsubjects.length : 0;
    this.listQuery = this.listSubjects;
    this.setlistPagination();
    if (response) {
      this.loadingData = false;
    }
  }
};
</script>
