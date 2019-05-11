<template>
  <div>
    <b-card>
      <div class="row">
        <div class="col-md-6">
          <div class="form-group">
            <label>Nama admin</label>
            <input type="text" class="form-control" v-model="name">
          </div>
          <div class="form-group">
            <label>Tanggal</label>
            <input v-model="date" class="form-control" type="date">  
          </div>  
          <div class="form-group">
            <label>Tipe Surat</label>
            <select v-model="selected" name="" id="" class="form-control">
                <option value="1">Masuk</option>
                <option value="0">Keluar</option>
            </select> 
          </div>  
        </div>
        <div class="col-md-6">
          <div class="form-group">
            <label>Kode Surat</label>
            <input v-model="code" type="text" class="form-control">
          </div>
          <div class="form-group">
            <label>No Tlp Client</label>
            <input v-model="phone" type="text" class="form-control">
          </div>
          <div class="form-group">
            <label>Nama Client</label>
            <input v-model="client" type="text" class="form-control">
          </div>
        </div>
      </div>
    </b-card>
    <b-card>
      <div>

        <button v-if="selected == 1" class="btn btn-success" style="margin-bottom:10px !important" @click="addNewItem">Tambah Item</button>
        <button v-else class="btn btn-success" style="margin-bottom:10px !important" @click="checkData">Cek Data</button>
        <div class="table-responsive">
          <table class="table">
            <thead>
              <tr>
                <th>No</th>
                <th>Nama Barang</th>
                <th>Qty</th>
                <th>Harga</th>
                <th>Total</th>
                <th>Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in items" v-bind:key="index">
                <td>{{ index = index + 1 }}</td>
                <td><input v-model="item.name" name="items[]['name']"  style="border:none !important" type="text" placeholder="Nama Barang"></td>
                <td><input v-model="item.qty" name="items[]['qty']"   style="border:none !important" type="number" placeholder="Qty"></td>
                <td><input v-model="item.price" name="items[]['price']"   style="border:none !important" type="text" placeholder="Harga"></td>
                <td><input v-model="item.total" name="items[]['total']"   readonly style="border:none !important" type="text" placeholder="Total"></td>                
                <td><button @click="remove(index)"  v-if="selected == 1" class="btn btn-danger btn-sm">Hapus</button></td>                
              </tr>            
            </tbody>
            <tfoot>
              <td></td>
              <td></td>
              <td></td>
              <td>Total Barang</td>
              <td>{{ total }}</td>
              <td></td>
            </tfoot>
          </table>
        </div>
      </div>
    </b-card>
    <b-card>
      <div>
        <div class="row">
          <div class="col-md-9"></div>
          <div class="col-md-3">
            <button class="btn btn-sm btn-danger btn-sm">Batal</button>
            <button class="btn btn-sm btn-success btn-sm" @click="postData">Simpan</button>
          </div>
        </div>
      </div>
    </b-card>
  </div>
</template>

<script>
import $ from 'jquery';
import axios from 'axios';
export default {
  
  mounted(){
    let self = this;
    // self.alert("Wisnu astri");
    
    $(function(){
      
    });
  },
  data(){
    return{
      name:'',
      mailname:'',
      date:'',
      code:'',
      base_url:'http://localhost:8000/api',
      selected:'',
      client:'',
      phone:'',
      total:0,
      items:[
        {
          name:'',
          qty:'',
          price:'',
          total:'',
        },
        
      ],    
    }
  },
  methods:{
   remove(index){
     console.log(index);
     let self = this;
     let newItems = [];
     delete self.items[index-1];
     
     self.items.forEach((value,indexVal)=>{
       if(index-1 != indexVal){
         newItems.push({
            name:value.name,
            qty:value.qty,
            price:value.price,
            total:value.total,
         })
       }
     });
     
     self.items = newItems;
   } ,
   addNewItem: function () {
      this.items.push({
          name:'',
          qty:'',
          price:'',
          total:'',
      });
    },
    checkData(){
        let self = this;
        axios(self.base_url+'/mail/check/'+self.code)
          .then((res)=>{
            if(res.data.status == 'error'){
              console.log("Data tidak ada");
            }else{
              self.items = res.data.data.items;
              console.log(res.data);
            }

          })
          .catch((err)=>{
            console.log(err);
          })
    },
    postData(){
     let self = this;
     if(self.selected == 1){
        axios({
            method: 'post',
            url: self.base_url+'/mail',
            data: {
              name: self.name,
              date: self.date,
              type: self.selected,
              code: self.code,
              phone:self.phone,
              client:self.client,
              items:self.items 
            }
          })
          .then((res)=>{
            self.items    = [];
            self.name     = null;
            self.date     = null;
            self.selected = null
            self.phone    = null;
            self.client   = null;
            this.$swal({
              type:'success',
              text:'Data berhasil di simpan'
            });
          })
          .catch((err)=>{
            console.log(err);
          });
     } else{
       //Delete
        axios({
            method: 'delete',
            url: self.base_url+'/mail/'+self.code+'/destroy',            
          })
          .then((res)=>{
            self.items    = [];
            self.name     = null;
            self.date     = null;
            self.selected = null
            self.phone    = null;
            self.client   = null;
            this.$swal({
              type:'success',
              text:'Data berhasil di hapus'
            });
            console.log(res);
          })
          .catch((err)=>{
            console.log(err);
          });
     }

    }
  },
  watch:{
   items: {
      handler: function (after, before,c) {
        let self = this;
        // Return the object that changed
        let totalAll = 0;
        after.forEach((value,index)=>{
          let qty = parseInt(value.qty);
          if(isNaN(qty)){
             qty = 0; 
          }else{
            let qty = parseInt(value.qty);
          }



          let price = parseInt(value.price);

          if(isNaN(price)){
            price = 0;  
          }else{
            let price = parseInt(value.price);
          }
          let total = qty * price;          
          value.total = total;
          totalAll = totalAll + total;
        })
        self.total = totalAll;
        // Log it
        // console.log(changed)
      },
      deep: true
    }
  },
  computed:{
    itemsProcess(){
      let self = this;
      let newArr = [];
      self.items.forEach((value,index)=>{
        let price = value.price;
        let qty = value.qty;
        let total = price * qty;
        newArr.push({
            name:value.name,
            qty:value.qty,
            price:value.price,
            total:total,
        });
      });
        return newArr;
    }
  }
}
</script>

