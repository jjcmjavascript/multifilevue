<template>
    <div class="row">

        <div class="form-group col-xs-12" :class="size" v-if="!config.cargando">
            <label> Archivos </label>
            <div class="input-group">
                <input :disabled="disabledUnique" type="file" class="form-control" :id="config.id" @input="support_file($event)" :accept="formatoValidos">
                <span class="input-group-btn" v-if="buttons[0]">
                    <button class="btn btn-success btn-flat text-center"  @click="addFile()" :disabled="disableAdd || disabledUnique">
                        <span v-html="confirm"> </span>
                    </button>
                </span>
            </div>
        </div>

        <div class="form-group col-xs-12" v-if="withType" :class="size">
            <label> Tipo Documento</label>
            <v-select v-model="tipo" :options="tipo_doc.tipo" :label="tipo_doc.label" />
        </div>

        <div class="form-group col-xs-12" >
            <div class="table-responsive" v-if="files.length > 0">
                <table class="col-xs-12 table-stripped table-bordered table-hover" >
                    <thead>
                        <tr>
                            <th>Nombre</th>
                            <th v-if="withType">Tipo Archivo</th>
                            <th>Tamaño</th>
                            <th v-if="buttons[1]"></th>
                        </tr>
                    </thead>
                    <tbody>
                        <template v-if="withType">
                            <tr v-for="(archivo, i) in files">
                                <td>{{archivo.file ? archivo.file.name : '' }}</td>
                                <td>{{archivo.tipo ? archivo.tipo[tipo_doc.label]: '' }}</td>
                                <td>{{getSize(archivo.file)}} Kb</td>
                                <td v-if="buttons[1]">
                                    <button class="btn btn-danger btn-flat btn-sm"  @click="borrar(i)">
                                        <span v-html="del"> </span>
                                    </button>
                                </td>
                            </tr>
                        </template>
                        <template v-else>
                            <tr v-for="(archivo, i) in files">
                                <td>{{archivo.name}}</td>
                                <td>{{ getSize(archivo) }} Kb</td>
                                <td v-if="buttons[1]" class="text-danger">
                                    <button class="btn btn-danger btn-flat btn-sm"  @click="borrar(i)">
                                        <i class="fa fa-trash"></i>
                                    </button>
                                </td>
                            </tr>
                        </template>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>

<script>

/************
    @param Obejct tipo_doc recibe dos valores el Array de Objetos ed los tipo de documento y El label para el V-Select
    @param Array extension array con las extensiones soportadas por el Input File
    @param String config HTML / Texto para nombre del agregar
    @param String del HTML / Texto para nombre del agregar
    @params String tamaaño,  Texto usando Bootrap3 para el damaño de inpout file
    @params Object buttons, booleano para mostrar o no el boton de eliminar

    PARA LIMPIAR EJECUTAR CLEAN USANDO REF
********/
export default {
    props: {
        value : {},
        buttons : {
            type: Array,
            default() {
                return [true, true]
            }
        },
        extension : {
            default : ()=> [],
            type : Array,
        },
        tipo_doc : {
            default : ()=> {
                return {
                    tipo : [],
                    label : 'nombre',
                }
            },
            type : Object,
        },
        confirm: {
            type : String,
            default : "<i class='fa fa-plus'> </i>"

        },
        del: {
            type : String,
            default : "<i class='fa fa-trash'> </i>"
        },
        size: {
            type : String,
            default : 'col-md-6',
        },
        unique : {
            type : Boolean,
            default : false,
        },
    },
    data(){
        return {
            config : {
                cargando : false, //sin uso aun
                id : null,
                support : false,
            },
            tipo : null,
            files : [

            ],
        }
    },

    methods : {
        support_file(event){
            if(event.target.files[0]){
                this.config.support = true;
            }
            else {
                this.config.support = false
            };
        },
        clean(){
            this.tipo = null;
            document.querySelector(`#${this.config.id}`).value = null;
            this.files = [];
            this.config.support = false;
        },
        generarID(){
            //para multiples instancias
           let result           = '';
           let characters       = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz';
           let charactersLength = characters.length;
           for ( let i = 0; i < 8; i++ ) {
              result += characters.charAt(Math.floor(Math.random() * charactersLength));
           }
            this.config.id =  result;
        },
        addFile(){
            let documento_activo = document.querySelector(`#${this.config.id}`);

            if( this.tipo_doc.tipo.length > 0 ){
                this.files.push({ file : documento_activo.files[0] , tipo : this.tipo });
            }
            else{
                this.files.push(documento_activo.files[0]);
            }
            // this.$emit('addFile');
            this.tipo = null;
            this.config.support = false;
            document.querySelector(`#${this.config.id}`).value = null;
            this.setModel();
        },
        getSize(archivo){
            if(archivo){
                return (archivo.size / 1024).toFixed(1);
            }
            return 0;
        },
        borrar(i){
            this.files.splice(i,1);
            this.setModel();
        },
        setModel(){
            this.$emit('input',this.files);
        },
        // funcion para setear el modelo desde el padre
        setModelInverse(files){
            this.files = files;
        },
    },
    computed : {
        disabledUnique() {
            return this.unique && this.files.length > 0;
        },
        disableAdd(){
            let doc = document.querySelector(`#${this.config.id}`);

            if(this.withType && this.tipo && doc && doc.files[0] ){
                return false;
            }if( !this.withType && this.config.support ){
                return false;
            }

            return true;
        },
        withType(){
            if( this.tipo_doc.tipo.length > 0 ){
                return true
            }
            return false;
        },
        formatoValidos(){
            if( this.extension.length > 0 ){
                let validos = this.extension.map(e => {return `.${e}`});
                return validos.join(',');
            }
            return '.*';
        },

    },
    mounted () {
        this.generarID()
    },
}
</script>

<style lang="css" scoped>
</style>
