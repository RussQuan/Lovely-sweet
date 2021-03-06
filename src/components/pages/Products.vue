<template>
  <div>
    <loading :active.sync="isLoading" :is-full-page="true">
      <template slot="default">
        <i class="fas fa-spinner fa-pulse fa-5x" style="color:#F1B1C3;"></i>
      </template>
    </loading>
    <div class="text-right">
      <button type="button" class="btn btn-secondary my-2" @click="openModal(true)">建立新的產品</button>
    </div>
    <div class="table-responsive text-nowrap">
      <table class="table mt-2">
        <thead>
          <th width="100">分類</th>
          <th>名稱</th>
          <th width="100">原價</th>
          <th width="100">售價</th>
          <th width="120">是否啟用</th>
          <th width="130">操作</th>
        </thead>
        <tbody v-for="item in products" :key="item.id">
          <td>{{ item.category }}</td>
          <td>{{ item.title }}</td>
          <td class="text-right">{{ item.origin_price | currency }}</td>
          <td class="text-right">{{ item.price | currency }}</td>
          <td>
            <span class="text-success" v-if="item.is_enabled == 1">啟用</span>
            <span class="text-success text-danger" v-else>未啟用</span>
          </td>
          <td>
            <button class="btn btn-outline-primary btn-sm" @click="openModal(false, item)">編輯</button>
            <button class="btn btn-outline-danger btn-sm" @click="delProductModal(item)">刪除</button>
          </td>
        </tbody>
      </table>
    </div>

    <PaginationBack :pages="pagination" @emitPages="getProducts"></PaginationBack>

    <div
      class="modal fade"
      id="productModal"
      tabindex="-1"
      role="dialog"
      aria-labelledby="exampleModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog modal-lg" role="document">
        <div class="modal-content border-0">
          <div class="modal-header bg-pinkdark text-white">
            <h5 class="modal-title" id="exampleModalLabel">
              <span>新增產品</span>
            </h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <div class="row">
              <div class="col-sm-4">
                <div class="form-group">
                  <label for="image">輸入圖片網址</label>
                  <input
                    type="text"
                    class="form-control"
                    id="image"
                    placeholder="請輸入圖片連結"
                    v-model="tempProduct.imageUrl"
                  />
                </div>
                <div class="form-group">
                  <label for="customFile">
                    或 上傳圖片
                    <i
                      class="fas fa-spinner fa-pulse"
                      style="color:#F1B1C3;"
                      v-if="status.fileUpLoading"
                    ></i>
                  </label>
                  <input
                    type="file"
                    id="customFile"
                    class="form-control"
                    ref="files"
                    @change="uploadFile()"
                  />
                </div>
                <img class="img-fluid" :src="tempProduct.imageUrl" alt />
              </div>
              <div class="col-sm-8">
                <div class="form-group">
                  <label for="title">標題</label>
                  <input
                    type="text"
                    class="form-control"
                    id="title"
                    placeholder="請輸入標題"
                    maxlength="60"
                    v-model="tempProduct.title"
                  />
                </div>

                <div class="form-row">
                  <div class="form-group col-md-6">
                    <label for="category">分類</label>
                    <input
                      type="text"
                      class="form-control"
                      id="category"
                      placeholder="請輸入分類"
                      maxlength="5"
                      v-model="tempProduct.category"
                    />
                  </div>
                  <div class="form-group col-md-6">
                    <label for="price">單位</label>
                    <input
                      type="unit"
                      class="form-control"
                      id="unit"
                      placeholder="請輸入單位"
                      maxlength="3"
                      v-model="tempProduct.unit"
                    />
                  </div>
                </div>

                <div class="form-row">
                  <div class="form-group col-md-6">
                    <label for="origin_price">原價</label>
                    <input
                      type="number"
                      class="form-control"
                      id="origin_price"
                      placeholder="請輸入原價"
                      v-model="tempProduct.origin_price"
                    />
                  </div>
                  <div class="form-group col-md-6">
                    <label for="price">售價</label>
                    <input
                      type="number"
                      class="form-control"
                      id="price"
                      placeholder="請輸入售價"
                      v-model="tempProduct.price"
                    />
                  </div>
                </div>
                <hr />

                <div class="form-group">
                  <label for="description">產品描述</label>
                  <textarea
                    type="text"
                    class="form-control"
                    id="description"
                    placeholder="請輸入產品描述"
                    v-model="tempProduct.description"
                  ></textarea>
                </div>
                <div class="form-group">
                  <label for="content">說明內容</label>
                  <textarea
                    type="text"
                    class="form-control"
                    id="content"
                    placeholder="請輸入產品說明內容"
                    v-model="tempProduct.content"
                  ></textarea>
                </div>
                <div class="form-group">
                  <div class="form-check">
                    <input
                      class="form-check-input"
                      type="checkbox"
                      id="is_enabled"
                      v-model="tempProduct.is_enabled"
                      :true-value="1"
                      :false-value="0"
                    />
                    <label class="form-check-label" for="is_enabled">是否啟用</label>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-outline-secondary" data-dismiss="modal">取消</button>
            <button type="button" class="btn btn-pinkdark text-white" @click="updateProduct">確認</button>
          </div>
        </div>
      </div>
    </div>
    <DelModal id="delProductModal" :modalData="tempProduct" @delCheck="delProduct"></DelModal>
  </div>
</template>

<script>
import $ from "jquery";
import PaginationBack from "../PaginationBack";
import DelModal from "../DelModal";

export default {
  data() {
    return {
      products: [],
      tempProduct: {},
      pagination: {},
      isNew: false,
      isLoading: false,
      status: {
        fileUpLoading: false
      }
    };
  },
  components: {
    PaginationBack,
    DelModal,
  },
  methods: {
    getProducts(page = 1) {
      const vm = this;
      vm.isLoading = true;
      const api = `${process.env.APIPATH}/api/${process.env.CUSTOMERPATH}/admin/products?page=${page}`;
      this.$http.get(api).then(response => {
        if (response.data.success) {
          console.log(response.data);
          vm.products = response.data.products;
          vm.pagination = response.data.pagination;
          vm.isLoading = false;
        }
      });
    },
    openModal(isNew, item) {
      if (isNew) {
        this.tempProduct = {};
        this.isNew = true;
      } else {
        this.tempProduct = Object.assign({}, item);
        this.isNew = false;
      }
      $("#productModal").modal("show");
    },
    updateProduct() {
      const vm = this;
      let api = `${process.env.APIPATH}/api/${process.env.CUSTOMERPATH}/admin/product`;
      let httpMethod = "post";
      if (!vm.isNew) {
        api = `${process.env.APIPATH}/api/${process.env.CUSTOMERPATH}/admin/product/${vm.tempProduct.id}`;
        httpMethod = "put";
      }
      this.$http[httpMethod](api, { data: vm.tempProduct }).then(response => {
        if (response.data.success) {
          console.log(response.data);
          $("#productModal").modal("hide");
          vm.getProducts((vm.page = vm.pagination.current_page));
          this.$bus.$emit("messsage:push", response.data.message, "success");
        } else {
          $("#productModal").modal("hide");
          vm.getProducts((vm.page = vm.pagination.current_page));
          this.$bus.$emit("messsage:push", response.data.message, "danger");
        }
      });
    },
    delProductModal(item) {
      const vm = this;
      $("#delProductModal").modal("show");
      vm.tempProduct = Object.assign({}, item);
    },
    delProduct() {
      const vm = this;
      vm.isLoading = true;
      const api = `${process.env.APIPATH}/api/${process.env.CUSTOMERPATH}/admin/product/${vm.tempProduct.id}`;
      this.$http.delete(api).then(response => {
        if (response.data.success) {
          console.log(response.data);
          $("#delProductModal").modal("hide");
          vm.isLoading = false;
          vm.getProducts();
          this.$bus.$emit("messsage:push", response.data.message, "success");
        } else {
          vm.getProducts();
          this.$bus.$emit("messsage:push", response.data.message, "danger");
        }
      });
    },
    uploadFile() {
      const vm = this;
      vm.status.fileUpLoading = true;
      const uploadedFile = this.$refs.files.files[0];
      const url = `${process.env.APIPATH}/api/${process.env.CUSTOMERPATH}/admin/upload`;
      const formData = new FormData();
      formData.append("file-to-upload", uploadedFile);
      this.$http
        .post(url, formData, {
          headers: {
            "Content-type": "multipart/form-data"
          }
        })
        .then(response => {
          console.log(response.data);
          if (response.data.success) {
            vm.$set(vm.tempProduct, "imageUrl", response.data.imageUrl);
            vm.status.fileUpLoading = false;
          }
        });
    }
  },
  created() {
    this.getProducts();
  }
};
</script>