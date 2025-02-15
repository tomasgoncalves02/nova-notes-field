<template lang="html">
    <div class="flex flex-col space-y-2">
        <div v-if="reply_to_id != ''" class="flex space-x-2">
            <p class="self-center font-semibold">Replying to {{ reply_to_name }}</p>
            <span @click.stop="emitCancelReply" class="self-center cursor-pointer dim">
                <x-circle />
            </span>
        </div>

        <div class="flex flex-col">
            <div class="flex flex-col space-y-2 " v-if="errors.length > 0">
                <p class="text-red-500" v-for="error in errors">{{ error }}</p>
            </div>
            <div :class="[richText ? 'space-y-4' : 'space-x-4', 'mt-1 flex flex-wrap flex-row']">
                <div v-if="richText" class="w-full" >
                    <trix-note v-model="newNote"></trix-note>
                </div>
                <input-field v-else
                    class="flex-1"
                    v-model="newNote"
                    @enter="onSubmit"
                    :hasError="errors.length > 0"
                ></input-field>
                <span @click="onSubmit" :class="[richText ? 'order-last ml-auto' : '','flex-shrink-0 justify-self-center self-center cursor-pointer']" @mouseover="isHovered=true"  @mouseleave="isHovered=false">
                        <icon-paper-airplane v-if="!isHovered"/>
                        <icon-paper-airplane-solid v-else/>
                </span>
                <div :class="[{'w-full justify-items-start p-0':!richText}, 'flex space-x-2 my-2']">
                    <toggle v-model="isPrivate">
                        <template #iconOff>
                            <icon-eye-solid />
                        </template>
                        <template #iconOn>
                            <icon-eye-off />
                        </template>
                    </toggle>
                    <toggle v-model="richText">
                        <template #iconOff>
                            <icon-doc-text-solid />
                        </template>
                        <template #iconOn>
                            <icon-doc-text />
                        </template>
                    </toggle>
                </div>

            </div>

        </div>
    </div>
</template>

<script>
import Toggle from './Toggle.vue';
import InputField from './InputField.vue';
import TrixNote from './TrixNote.vue';
import IconEyeSolid from '../icons/IconEyeSolid.vue';
import IconEyeOff from '../icons/IconEyeOff.vue';
import IconDocTextSolid from '../icons/IconDocTextSolid.vue';
import IconDocText from '../icons/IconDocText.vue';
import XCircle from '../icons/XCircle.vue';
import IconPaperAirplane from '../icons/IconPaperAirplane.vue';
import IconPaperAirplaneSolid from '../icons/IconPaperAirplaneSolid.vue';

export default {
    components: {
        Toggle,
        InputField,
        TrixNote,
        IconEyeOff,
        IconEyeSolid,
        IconDocText,
        IconDocTextSolid,
        XCircle,
        IconPaperAirplane,
        IconPaperAirplaneSolid,
    },
    props: {
        notable_id: {
            type: Number,
            required: true
        },

        notable_type: {
            type: String,
            required: true
        },
        reply_to_id: {
            type: Number
        },
        reply_to_name: {
            type: String
        }
    },
    data() {
        return {
            isPrivate: false,
            richText: false,
            newNote: '',
            notesCount: 0,
            errors: [],
            isHovered: false
        };
    },
    methods: {
        onSubmit() {
            this.errors = [];
            Nova.request()
                .post('/nova-vendor/notes-field/new', {
                    notable_id: this.notable_id,
                    notable_type: this.notable_type,
                    note: this.newNote,
                    personal: this.isPrivate,
                    as_html: this.richText,
                    reply_to_id: this.reply_to_id
                })
                .then(({ data }) => {
                    data.reply_to_id = this.reply_to_id ? this.reply_to_id : undefined;
                    this.$emit('note-submit', data);
                    this.newNote = '';
                })
                .catch(e => {
                    if (e.response.data.errors.note) {
                        this.errors = e.response.data.errors.note;
                    }

                });

        },
        emitCancelReply() {
            this.$emit('cancel-reply')
        }
    },
    watch: {
        richText(usingTrix) {
            if(!usingTrix) {
                this.newNote = this.newNote.replace(/(<([^>]+)>)/gi, "");
            }
        }
    }
};
</script>

<style scoped src="../../../sass/field.css"></style>
