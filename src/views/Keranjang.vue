<template>
  <div class="keranjang">
    <Navbar />
    <div class="container">
      <!--Breadcrumb-->
      <div class="row mt-4">
        <div class="col">
          <nav aria-label="breadcrumb">
            <ol class="breadcrumb">
              <li class="breadcrumb-item">
                <router-link to="/" class="text-dark">Home</router-link>
              </li>
              <li class="breadcrumb-item">
                <router-link to="/foods" class="text-dark">Foods</router-link>
              </li>
              <li class="breadcrumb-item active" aria-current="page">
                Keranjang
              </li>
            </ol>
          </nav>
        </div>
      </div>

      <div class="row">
        <div class="col">
          <h2>Keranjang <strong>Saya</strong></h2>
          <div class="table-responsiv mt-3">
            <table class="table">
              <thead>
                <tr>
                  <th scope="col">#</th>
                  <th scope="col">Gambar</th>
                  <th scope="col">Makanan</th>
                  <th scope="col">Keterangan</th>
                  <th scope="col">Jumlah</th>
                  <th scope="col">Harga</th>
                  <th scope="col">Total Harga</th>
                  <th scope="col">Hapus</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(keranjang, index) in keranjang" :key="keranjang.id">
                  <th>{{ index + 1 }}</th>
                  <td>
                    <img
                      :src="
                        require('../assets/images/' + keranjang.products.gambar)
                      "
                      class="img-fluid shadow"
                      width="250"
                    />
                  </td>
                  <td>
                    <strong>{{ keranjang.products.nama }}</strong>
                  </td>
                  <td>
                    {{ keranjang.keterangan ? keranjang.keterangan : "-" }}
                  </td>
                  <td>
                    {{ keranjang.jumlah_pemesanan }}
                  </td>
                  <td align="right">Rp. {{ keranjang.products.harga }}</td>
                  <td align="right">
                    <strong>
                      Rp.
                      {{
                        keranjang.products.harga * keranjang.jumlah_pemesanan
                      }}</strong
                    >
                  </td>
                  <td align="center" class="text-danger">
                    <b-icon-trash
                      @click="hapusKeranjang(keranjang.id)"
                    ></b-icon-trash>
                  </td>
                </tr>

                <tr>
                  <td colspan="6" align="right">
                    <strong>Total Harga :</strong>
                  </td>
                  <td align="right">
                    <strong>Rp. {{ totalHarga }}</strong>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
      <div class="row justify-content-end">
        <div class="col-md-4">
          <form class="mt-4" v-on:submit.prevent>
            <div class="form-group">
              <label for="nama">Nama :</label>
              <input type="text" class="form-control" v-model="pesan.nama" />
            </div>
            <div class="form-group">
              <label for="noMeja">Nomor Meja :</label>
              <input
                type="number"
                class="form-control"
                v-model="pesan.noMeja"
              />
            </div>

            <button
              type="submit"
              class="btn btn-success float-right"
              @click="checkout"
            >
              <b-icon-cart></b-icon-cart>Pesan
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Navbar from "@/components/Navbar.vue";
import axios from "axios";

export default {
  name: "Keranjang",
  components: {
    Navbar,
  },
  data() {
    return {
      keranjang: [],
      pesan: {},
    };
  },
  methods: {
    setKeranjangs(data) {
      this.keranjang = data;
    },
    hapusKeranjang(id) {
      axios
        .delete(`http://localhost:3000/keranjang/${id}`)
        .then(() => {
          this.updateKeranjang(); // Update data setelah penghapusan
          this.$toast.success("Item berhasil dihapus dari keranjang", {
            type: "success",
            position: "top-right",
            duration: 3000,
            dismissible: true,
          });
        })
        .catch((error) => {
          console.error(error);
        });
    },
    updateKeranjang() {
      axios
        .get(`http://localhost:3000/keranjang/`)
        .then((response) => {
          this.setKeranjangs(response.data);
          this.$toast.success("Data keranjang berhasil diperbarui", {
            type: "success",
            position: "top-right",
            duration: 3000,
            dismissible: true,
          });
        })
        .catch((error) => {
          console.error(error);
        });
    },
    checkout() {
      if (this.pesan.nama && this.pesan.noMeja) {
        this.pesan.keranjang = this.keranjang;
        axios
          .post("http://localhost:3000/pesanan", this.pesan)
          .then(() => {

            //hapus semua keranjang
            this.keranjang.map(function (item) {
              return axios
                .delete("http://localhost:3000/keranjang/" + item.id)
                .catch((error) => console.log(error));
            });
            this.$router.push({ path: "/pesanan-sukses" });
            this.$toast.success("Sukses Dipesan", {
              type: "success",
              position: "top-right",
              duration: 3000,
              dismissible: true,
            });
          })
          .catch((err) => {
            console.error("Terjadi kesalahan saat mengirim data:", err);
          });
      } else {
        this.$toast.error("Nama dan Nomor Meja Harus Di Isi", {
          type: "error",
          position: "top-right",
          duration: 3000,
          dismissible: true,
        });
      }
    },
  },
  mounted() {
    this.updateKeranjang(); // Load data keranjang saat komponen dimuat
  },
  computed: {
    totalHarga() {
      return this.keranjang.reduce((total, item) => {
        return total + item.products.harga * item.jumlah_pemesanan;
      }, 0);
    },
  },
};
</script>

<style>
</style>







