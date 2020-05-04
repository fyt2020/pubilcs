<template>
  <div class="fDetalis-content-two">
    <div class="left-con">
      <div class="potho">
        <div class="same">
          <!-- <img :src="imgOne" alt id="one" class="img" /> -->
          <img :src="front" alt id="ONE" class="img" v-if="front" />
        </div>
        <div class="same">
          <!-- <img :src="imgOne" alt id="one" class="img" /> -->
          <img :src="left" alt id="TWO" class="img" v-if="left" />
        </div>
        <div class="same">
          <!-- <img :src="imgOne" alt id="one" class="img" /> -->
          <img :src="right" alt id="THR" class="img" v-if="right" />
        </div>
      </div>
      <p class="potho__info">
        <span class="potho__item">正面照</span>
        <span class="potho__item">左面照</span>
        <span class="potho__item">右面照</span>
      </p>
    </div>
  </div>
</template>

<script>
import * as URL from "@/plugins/env.js";
import jsZip from "jszip";
export default {
  data() {
    return {
      id: "",
      cen: "",

      front: "",
      left: "",
      right: "",

      imgOne: require("@/assets/img/cen.png")
    };
  },
  computed: {},
  created() {
    if (!this.$route.params.id) {
      this.id = JSON.parse(localStorage.getItem("id"));
    } else {
      this.id = this.$route.params.id;
    }
  },
  mounted() {
    // this.getData(this.id,this.getcent);
    this.getcent(this.id)
  },
  beforeDestroy() {
    this.front = "";
    this.left = "";
    this.right = "";
  },

  methods: {

    getData(url,succ,fail){
        let path = window.BASEURL.oss + `${url}_data/snap_${url}`;
        let xmlhttp = null;
        if (window.XMLHttpRequest) {
            xmlhttp = new window.XMLHttpRequest()
        } else {
            xmlhttp = new window.ActiveXObject('Microsoft.XMLHTTP')
        };
        xmlhttp.open('GET', path, true);
        // xmlhttp.addEventListener("progress", function (evt) {
        //     if (evt.lengthComputable) {
        //         var percentCompvare = evt.loaded / evt.total * 100;//加载程度百分比
        //         var elem = document.getElementById("myBar");
        //         var width = parseInt(percentCompvare);
        //         elem.style.width = width + '%';
        //         if(percentCompvare < 99 || percentCompvare === 99){
        //             $(".loading").html("正在下载:" + parseInt(percentCompvare) + "%");
        //         }
        //     }
        // }, false);
        xmlhttp.setRequestHeader("X-Auth-Token", "AUTH_d11d29fbaee74a379a694ce68396bc6a");
        xmlhttp.withCredentials = true;
        if ('responseType' in xmlhttp) {
            xmlhttp.responseType = 'arraybuffer'
        }
        if (xmlhttp.overrideMimeType) {
            xmlhttp.overrideMimeType('text/plain; charset=x-user-defined')
        }
        xmlhttp.send();
        xmlhttp.onreadystatechange = function(){
                console.log(xmlhttp.response )
                // console.log(xmlhttp.responseText )
            if (xmlhttp.readyState === 4 && xmlhttp.status === 200) {
                var file = xmlhttp.response || xmlhttp.responseText;
                succ(file)
            }
        }
    },







    getcent(data) {
      console.log(data)
      // let VM = this;
      // let file = data;
      // let Zip = new jsZip();
      //       Zip.loadAsync(file).then(function(zip) {
      //         zip.forEach(function(relativePath, zipEntry) {
      //           // console.log(zipEntry.name);
      //           if (
      //             zipEntry.name == "images/" ||
      //             zipEntry.name == "images-2/" ||
      //             zipEntry.name == "images-3/" ||
      //             zipEntry.name == "info.txt" ||
      //             zipEntry.name == "output/"
      //           )
      //             return;
      //           if (zipEntry.name.indexOf("output/front") > -1) {
      //             Zip.file(zipEntry.name)
      //               .async("base64")
      //               .then(function(content) {
      //                 VM.front = "data:image/jpeg;base64," + content;
      //               });
      //           }
      //           if (zipEntry.name.indexOf("output/left") > -1) {
      //             Zip.file(zipEntry.name)
      //               .async("base64")
      //               .then(function(content) {
      //                 VM.left = "data:image/jpeg;base64," + content;
      //               });
      //           }
      //           if (zipEntry.name.indexOf("output/right") > -1) {
      //             Zip.file(zipEntry.name)
      //               .async("base64")
      //               .then(function(content) {
      //                 VM.right = "data:image/jpeg;base64," + content;
      //               });
      //           }
      //         });
      //       });


      
      //解压图片
      let VM = this;
      this.$http.api
        .get(window.BASEURL.oss + `${data}_data/snap_${data}`, {
          headers: {
            "X-Auth-Token": "AUTH_d11d29fbaee74a379a694ce68396bc6a",
             "Content-Type": "text/plain;charset=utf-8"
          },
          timeout: 0,
          responseType: "arraybuffer"
        })
        .then(function(res) {
          console.log(res.data);
          if (res.status === 200) {
            let file = res.data;
            let Zip = new jsZip();
            Zip.loadAsync(file).then(function(zip) {
              zip.forEach(function(relativePath, zipEntry) {
                // console.log(zipEntry.name);
                if (
                  zipEntry.name == "images/" ||
                  zipEntry.name == "images-2/" ||
                  zipEntry.name == "images-3/" ||
                  zipEntry.name == "info.txt" ||
                  zipEntry.name == "output/"
                )
                  return;
                if (zipEntry.name.indexOf("output/front") > -1) {
                  Zip.file(zipEntry.name)
                    .async("base64")
                    .then(function(content) {
                      VM.front = "data:image/jpeg;base64," + content;
                    });
                }
                if (zipEntry.name.indexOf("output/left") > -1) {
                  Zip.file(zipEntry.name)
                    .async("base64")
                    .then(function(content) {
                      VM.left = "data:image/jpeg;base64," + content;
                    });
                }
                if (zipEntry.name.indexOf("output/right") > -1) {
                  Zip.file(zipEntry.name)
                    .async("base64")
                    .then(function(content) {
                      VM.right = "data:image/jpeg;base64," + content;
                    });
                }
              });
            });
          }
        })
        .catch(function(error) {
          VM.$Message.error(`not found`);
          console.log(error);
        });
    }
  },
  components: {}
};
</script>

<style scoped lang="scss">
@import "@/assets/css/func.scss";
.fDetalis-content-two {
  width: 100%;
  height: 100%;
  position: relative;
  display: flex;
  padding: 0 px2Rem(20px);
  @media screen and (max-width: 1366px) {
    margin-top: 20px;
  }
  .left-con {
    width: 100%;
    display: flex;
    flex-direction: column;

    @media screen and (max-width: 1280px) {
      justify-content: center;
      align-items: center;
    }
    .potho {
      width: 100%;
      display: flex;
      justify-content: flex-start;
      @media screen and (max-width: 1280px) {
        justify-content: center;
        align-items: center;
      }
      .same {
        width: px2Rem(330px);
        height: px2Rem(430px);
        display: flex;
        flex-direction: column;
        align-items: center;
        margin-right: px2Rem(60px);
        background: #b1b1b5;
        border-radius: 6px;
        border: solid 1px #dcdfe6;
        box-shadow: 0px 5px 14px 2px rgba(0, 0, 0, 0.1);
        @media screen and (max-width: 1366px) {
          width: 220px;
          height: 286px;
        }
        @media screen and (max-width: 1280px) {
          margin-left: 30px;
        }
        .img {
          width: 100%;
          height: 100%;
          border-radius: 6px;
        }
      }
    }
    .potho__info {
      display: flex;
      justify-content: flex-start;
      width: 100%;
      @media screen and (max-width: 1280px) {
        justify-content: center;
        align-items: center;
      }
      .potho__item {
        width: px2Rem(330px);
        display: flex;
        flex-direction: column;
        align-items: center;
        margin-right: px2Rem(60px);
        color: #252a2a;
        font-size: px2Rem(20px);
        margin-top: 20px;
        @media screen and (max-width: 1366px) {
          width: 220px;
        }
        @media screen and (max-width: 1280px) {
            margin-right: 0;
            font-size:20px;
            margin-right: 40px;
            margin-left: 30px;

        
      }
      }
    }
  }
}
</style>
