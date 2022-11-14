<template>
    <div class="ww-video-weweb" :class="{ editing: isEditing }">
        <video ref="videoPlayer" class="video-element" playsinline webkit-playsinline v-bind="videoAttributes">
            Sorry, your browser doesn't support embedded videos.
        </video>
    </div>
</template>

<script>
export default {
    props: {
        content: { type: Object, required: true },
        uid: { type: String, required: true },
        /* wwEditor:start */
        wwEditorState: { type: Object, required: true },
        /* wwEditor:end */
    },
    emits: ['update:sidepanel-content', 'trigger-event'],
    setup(props) {
        const player = null;
        const { variableValue: isPlayingVariableValue, setValue: setIsPlayingValue } =
            wwLib.wwVariable.useComponentVariable({
                uid: props.uid,
                name: 'Is Playing',
                type: 'boolean',
                defaultValue: false,
                readonly: true,
            });
        const { variableValue: currentTimeVariableValue, setValue: setCurrentTimeValue } =
            wwLib.wwVariable.useComponentVariable({
                uid: props.uid,
                name: 'Current time',
                type: 'number',
                defaultValue: 0,
                readonly: true,
            });

        return { player, isPlayingVariableValue, setIsPlayingValue, currentTimeVariableValue, setCurrentTimeValue };
    },
    computed: {
        isEditing() {
            /* wwEditor:start */
            return this.wwEditorState.editMode === wwLib.wwEditorHelper.EDIT_MODES.EDITION;
            /* wwEditor:end */
            // eslint-disable-next-line no-unreachable
            return false;
        },
        isPreviewImageWeWeb() {
            return this.content.previewImage && this.content.previewImage.startsWith('designs/');
        },
        previewImageSrc() {
            return this.isPreviewImageWeWeb
                ? `${wwLib.wwUtils.getCdnPrefix()}${this.content.previewImage}`
                : this.content.previewImage;
        },
        videoAttributes() {
            const attributes = {
                src: this.content.file,
                poster: this.previewImageSrc,
                muted: this.content.muted,
            };

            if (this.content.autoplay) attributes.autoplay = this.isEditing ? false : true;
            if (this.content.muted) attributes.muted = true;
            if (this.content.controls) attributes.controls = true;
            if (this.content.loop) attributes.loop = true;
            if (this.content.preload) attributes.preload = true;

            return attributes;
        },
    },
    watch: {
        isEditing() {
            this.initVideo();
        },
        'content.file'() {
            this.initVideo();
        },
        'content.autoplay'(value) {
            if (!this.content.muted) this.$emit('update:content', { muted: true });
            if (this.player && value && !this.isEditing) this.player.play();
        },
    },
    methods: {
        initVideo() {
            this.player = this.$refs.videoPlayer;
            if (!this.player) return;

            this.player.pause();
            this.player.currentTime = Math.ceil(this.content.videoStartTime);

            if (this.isEditing) return;

            /* wwEditor:start */
            if (typeof this.player.duration == 'number') {
                this.$emit('update:sidepanel-content', {
                    path: 'videoDuration',
                    value: Math.ceil(this.player.duration),
                });
            }
            /* wwEditor:end */

            this.player.ontimeupdate = event => {
                this.updateCurrentTime(event.target.currentTime);
            };
            this.player.onplay = () => this.updateIsPlaying(true);
            this.player.onpause = () => this.updateIsPlaying(false);
            this.player.onended = () =>
                this.$emit('trigger-event', { name: 'end', event: { value: this.player.currentTime } });

            if (this.content.autoplay) this.player.play();
        },
        updateCurrentTime(currentTime) {
            if (typeof currentTime !== 'number') return;
            this.setCurrentTimeValue(currentTime.toFixed(2));
        },
        updateIsPlaying(isPlaying) {
            this.setIsPlayingValue(isPlaying);
            if (isPlaying) {
                this.$emit('trigger-event', { name: 'play', event: { value: this.player.currentTime } });
            } else {
                this.$emit('trigger-event', { name: 'pause', event: { value: this.player.currentTime } });
            }
        },
    },
};
</script>

<style lang="scss">
.ww-video-weweb {
    position: relative;
    overflow: hidden;

    &.editing {
        pointer-events: none;
    }

    .video-element {
        position: absolute;
        top: 0;
        left: 0;
        bottom: 0;
        right: 0;
        width: 100%;
        height: 100%;
    }
}
</style>
