<template>
		<div ref="container" class="video-input">			<div :style="estilo_contenedor" class="contenedor">				<canvas ref="preview" v-on:dragenter.stop.prevent="dragenter_stop_prevent_ID_93557390($event)" v-on:dragleave.stop.prevent="dragleave_stop_prevent_ID_863115416($event)" v-on:drag.stop.prevent="drag_stop_prevent_ID_1603140729($event)" v-on:dragover.stop.prevent="dragover_stop_prevent_ID_172012338($event)" v-on:dragstart.stop.prevent="dragstart_stop_prevent_ID_337767301($event)" v-on:dragend.stop.prevent="dragend_stop_prevent_ID_790769616($event)" v-on:keyup.enter="keyup_enter_ID_693230690($event)" v-on:drop.stop.prevent="cargar_video($event)" v-on:click.prevent="click_prevent_ID_1600585618($event)" :style="estilo" class="video-preview">
				</canvas>
					<div v-if="tmp.elegida == false" :style="estilo_texto" class="video-inner">								<div class="text-uppercase video-inner-text">{{ mensaje }}</div>
					</div>
				<input ref="fileInput" v-on:change="cargar_video($event)" :accept="tipo" style="display:none" name="archivo" type="file">
				</input>
			</div>
			<video ref="video" width="1" muted autoplay>
			</video>
		</div>
</template>

<script>
import daycaca from 'daycaca'
import _ from 'underscore'
import gifjs from 'gif.js'

export default {
	props: {default: { default: '' },mensaje: { default: 'CARGAR VIDEO' },radio: { default: 0 },ancho: { default: 608 },alto: { default: 342 },tipo: { default: 'video/*' },duracion: { default: -1 },ratio: { default: '16/9' },maxmb: { default: 20 },preview_ancho: { default: 96 },preview_alto: { default: 54 }},
	data() {
		return {
			tmp: {
				gif_frame: 0,
				gif_creado: 0,
				error: false,
				elegida: false,
				archivo: '',
				arrastrado_en_canvas: false
			},
			info: {
				ratio: '0/0',
				alto: 0,
				ancho: 0,
				bytes: 0,
				duracion: 0,
				preview: ''
			}
		};
	},
	mounted() {
				let gif = new gifjs({ workers: 2, quality: 10, width:this.preview_ancho, height:this.preview_alto });
		this.tmp = _.extend(this.tmp,{
			gif_obj : gif
		});
		this.$refs.video.addEventListener('play', function() {
   //console.log('reproduciendo video en canvas');
   this.play();
}.bind(this));
function gcd(u, v) {
   if (u === v) return u;
   if (u === 0) return v;
   if (v === 0) return u;

   if (~u & 1)
      if (v & 1)
         return gcd(u >> 1, v);
      else
         return gcd(u >> 1, v >> 1) << 1;

   if (~v & 1) return gcd(u, v >> 1);

   if (u > v) return gcd((u - v) >> 1, v);

   return gcd((v - u) >> 1, u);
}
/* returns x/y string with the ratio */
function ratio(w, h) {
   var d = gcd(w, h);
   return w / d + '/' + h / d;
}
this.$refs.video.addEventListener('loadedmetadata', function() {
   this.info.duracion = Math.round(this.$refs.video.duration);
   this.info.ancho = this.$refs.video.videoWidth;
   this.info.alto = this.$refs.video.videoHeight;
   this.info.bytes = this.tmp.archivo.size;
   this.info.ratio = ratio(this.info.ancho,this.info.alto);
   //obtenemos base64 de video
   let reader = new FileReader();
   reader.onload = e => {
      this.$emit('info',{ 
         duracion: this.info.duracion,
         ancho: this.info.ancho,
         alto: this.info.alto,
         bytes: this.tmp.archivo.size,
         mbytes: Math.round(this.tmp.archivo.size/1024/1024),
         ratio: this.info.ratio,
         base64: e.target.result
      });
      // revisamos si ratio o maxmb no se cumplen
      if (Math.round(this.tmp.archivo.size/1024/1024)>this.maxmb) {
         this.$emit('error',{ tipo:'tamano', mensaje:'MB superados' });
         this.tmp.error = true;
      } else if (this.info.ratio != this.ratio) {
         this.$emit('error',{ tipo:'ratio', mensaje:'Ratio incorrecto' });
         this.tmp.error = true;
      } else if (this.duracion != -1 && this.duracion!=this.info.duracion) {
         this.$emit('error',{ tipo:'duracion', mensaje:'Duracion incorrecta' });
         this.tmp.error = true;
      }
   }
   reader.readAsDataURL(this.tmp.archivo); 
}.bind(this));
	},
	asyncComputed: {
		async estilo() {
			var resp = {
height : this.alto + 'px',
width : this.ancho + 'px'};
			if (this.tmp.arrastrado_en_canvas==true||this.tmp.arrastrado_en_canvas=='true') {
				resp = _.extend(resp,{
					filter : 'brightness(0.5)'
				});
			}
			return resp;
		},
		async estilo_texto() {
			var resp = {
top : -this.alto+'px',
marginBottom : -this.alto+'px',
fontSize : '21px',
borderRadius : this.radio+'%',
zIndex : 10003};
			return resp;
		},
		async estilo_contenedor() {
			var resp = {
maxWidth : this.ancho+'px',
height : this.alto+'px',
borderRadius : this.radio+'%'};
			return resp;
		}
	},
	methods: {
		play: async function() {
			let reproduciendose = !((this.$refs.video && this.$refs.video.paused) || (this.$refs.video && this.$refs.video.ended));
		if (reproduciendose==true||reproduciendose=='true') {
			this.$refs.preview.getContext('2d').drawImage(this.$refs.video, 0, 0, this.$refs.preview.clientWidth/2, this.$refs.preview.clientHeight/2);
			if (this.tmp.error==false||this.tmp.error=='false') {
				if (this.tmp.gif_creado==0||this.tmp.gif_creado=='0') {
					this.$refs.video.currentTime=0
					this.tmp.gif_obj = new gifjs({ workers: 2, quality: 10, width:this.preview_ancho, height:this.preview_alto });
					this.tmp = _.extend(this.tmp,{
						gif_creado : 1
					});
					this.$emit('preview',{
						creando : true
					});
				}
				 else if (this.tmp.gif_creado==1||this.tmp.gif_creado=='1') {
					let cada_x = (this.tmp.gif_frame % 9 == 0);
					if (cada_x==true||cada_x=='true') {
						// clonamos canvas a uno temporal para hacer version Gif mini
let newCanvas = document.createElement('canvas');
let newContext = newCanvas.getContext('2d');
newCanvas.width = this.preview_ancho;
newCanvas.height = this.preview_alto;
newContext.drawImage(this.$refs.preview, 0, 0, this.preview_ancho, this.preview_alto);
// agregamos frame a GIF
this.tmp.gif_obj.addFrame(newContext, {
   copy: true,
   delay: 150
});
newContext = null;
newCanvas = null;
					}
					this.tmp = _.extend(this.tmp,{
						gif_frame : this.tmp.gif_frame+1
					});
				}			}
			setTimeout(this.play,20);
		}
 else {
			if (this.tmp.gif_creado==1||this.tmp.gif_creado=='1') {
				this.tmp = _.extend(this.tmp,{
					gif_creado : 2,
					gif_frame : 0
				});
			}
			if (this.tmp.error==false||this.tmp.error=='false') {
				this.tmp.gif_obj.on('finished', function(blob) {
   daycaca.base64(blob, (image) => {
      this.info.preview=image;
      //volvemos a reproducir video
      this.$refs.video.play();
      // avisamos a contenedor que esta listo preview
      this.$emit('preview', { creando:false, preview: image });
   });
}.bind(this));
try {
   this.tmp.gif_obj.render();
} catch(eer) {
   //volvemos a reproducir video
   this.$refs.video.play();
}
			}
 else {
				this.$refs.video.play()
}}
		},
		cargar_video: async function(e) {
			this.tmp = _.extend(this.tmp,{
			arrastrado_en_canvas : false
		});
		let archivos = e.target.files || e.dataTransfer.files
		if (archivos && archivos.length) {
			console.log('verificamos tamano de archivo',{"archivos":archivos});
			let valido = this.$refs.video.canPlayType(archivos[0].type);
			if (valido=='yes') {
				let tmp = URL.createObjectURL(archivos[0]);
				this.$refs.video.src = tmp;
				this.tmp = _.extend(this.tmp,{
					archivo : archivos[0],
					elegida : true,
					gif_creado : 0,
					gif_frame : 0,
					error : false
				});
				this.$emit('change',{
					archivo : archivos[0]
				});
			}
			 else if (valido=='maybe') {
				let tmp = URL.createObjectURL(archivos[0]);
				this.$refs.video.src = tmp;
				this.tmp = _.extend(this.tmp,{
					archivo : archivos[0],
					elegida : true,
					gif_creado : 0,
					gif_frame : 0,
					error : false
				});
				this.$emit('change',{
					archivo : archivos[0]
				});
			} else {
				this.$emit('error',{
					tipo : 'formato',
					mensaje : 'Formato invalido'
				});
}		}
		},
		dragenter_stop_prevent_ID_93557390: async function() {
			console.log('onDragEnter',{});
					this.tmp = _.extend(this.tmp,{
						arrastrado_en_canvas : true
					});
		},
		dragleave_stop_prevent_ID_863115416: async function() {
			console.log('onDragLeave',{});
					this.tmp = _.extend(this.tmp,{
						arrastrado_en_canvas : false
					});
		},
		drag_stop_prevent_ID_1603140729: async function() {
			
		},
		dragover_stop_prevent_ID_172012338: async function() {
			
		},
		dragstart_stop_prevent_ID_337767301: async function() {
			
		},
		dragend_stop_prevent_ID_790769616: async function() {
			
		},
		keyup_enter_ID_693230690: async function() {
			
		},
		click_prevent_ID_1600585618: async function() {
			this.$refs.fileInput.click()
		},
	}

}
</script>

<style scoped>
.video-input {
	width: 100%;
	margin: 0 auto;
	text-align: center;
}
.contenedor {
	width: 100%;
	box-sizing: border-box;
	margin: 0 auto;
	cursor: pointer;
	overflow: hidden;
}
.video-preview {
	width: 100%;
	height: 100%;
	position: relative;
	z-index: 10001;
	box-sizing: border-box;
	background-color: rgba(200, 200, 200, .25);
	image-rendering: optimizeSpeed;
	image-rendering: -moz-crisp-edges;
	image-rendering: -webkit-optimize-contrast;
	image-rendering: -o-crisp-edges;
	image-rendering: pixelated;
	-ms-interpolation-mode: nearest-neighbor;
}
.video-inner {
	position: relative;
	z-index: 10002;
	pointer-events: none;
	box-sizing: border-box;
	margin: 1em auto;
	padding: 0.5em;
	border: .3em dashed rgba(66, 66, 66, .15);
	border-radius: 8px;
	width: calc(100% - 2.5em);
	height: calc(100% - 2.5em);
	display: table;
}
.video-inner-text {
	display: table-cell;
	vertical-align: middle;
	text-align: center;
	font-size: 2em;
	line-height: 1.5;
}
</style>
