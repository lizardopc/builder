<template>
	<div class="relative flex items-center justify-between">
		<span
			class="inline-block text-[10px] font-medium uppercase text-gray-600 dark:text-zinc-400"
			:class="{
				'cursor-ns-resize': enableSlider,
			}"
			@mousedown="handleMouseDown">
			{{ label }}
		</span>
		<Input
			:type="type"
			placeholder="unset"
			:value="modelValue"
			:options="inputOptions"
			v-if="type != 'autocomplete'"
			@change="handleChange"
			@mousedown="handleMouseDown"
			:inputClass="type == 'checkbox' ? ' ml-2 !w-4' : 'pr-6'"
			class="rounded-md text-sm text-gray-800 dark:border-zinc-700 dark:bg-zinc-800 dark:text-zinc-200 dark:focus:bg-zinc-700"
			:class="{
				'w-[150px]': type != 'checkbox',
			}" />
		<Autocomplete
			v-if="type == 'autocomplete'"
			placeholder="unset"
			:modelValue="modelValue"
			:options="inputOptions"
			@update:modelValue="handleChange"
			class="!dark:text-zinc-200 !dark:focus:bg-zinc-700 w-[150px] rounded-md text-sm text-gray-800 dark:bg-zinc-800" />
		<div
			class="absolute right-1 top-[3px] cursor-pointer p-1 text-gray-700 dark:text-zinc-300"
			@click="clearValue"
			v-if="!['autocomplete', 'select', 'checkbox'].includes(type)"
			v-show="modelValue">
			<CrossIcon />
		</div>
	</div>
</template>
<script setup lang="ts">
import { isNumber } from "@tiptap/vue-3";
import { PropType, computed } from "vue";
import Autocomplete from "./Autocomplete.vue";
import CrossIcon from "./Icons/Cross.vue";

const props = defineProps({
	modelValue: {},
	label: {
		type: String,
		default: "",
	},
	type: {
		type: String,
		default: "text",
	},
	unitOptions: {
		type: Array as PropType<string[]>,
		default: () => [],
	},
	options: {
		type: Array,
		default: () => [],
	},
	enableSlider: {
		type: Boolean,
		default: false,
	},
	minValue: {
		type: Number,
		default: null,
	},
	maxValue: {
		type: Number,
		default: null,
	},
});

const emit = defineEmits(["update:modelValue"]);

type Option = {
	label: string;
	value: string;
};

const inputOptions = computed(() => {
	return (props.options || []).map((option) => {
		if (typeof option === "string" || (typeof option === "number" && props.type === "autocomplete")) {
			return {
				label: option,
				value: option,
			};
		}
		return option;
	}) as Option[];
});

// TODO: Refactor
const handleChange = (value: string | number | null | { label: string; value: string }) => {
	if (typeof value === "object" && value !== null && "value" in value) {
		value = value.value;
	}
	if (value && typeof value === "string") {
		let [_, number, unit] = value.match(/([0-9]+)([a-z%]*)/) || ["", "", ""];
		if (!unit && props.unitOptions.length && number) {
			value = number + props.unitOptions[0];
		}
	}

	emit("update:modelValue", value);
};

const handleMouseDown = (e: MouseEvent) => {
	if (!props.enableSlider) {
		return;
	}
	const value = (props.modelValue as string) || "";

	let [_, number, unit] = value.match(/([0-9]+)([a-z%]*)/) || ["", "", ""];

	if (!unit && props.unitOptions.length && number) {
		unit = props.unitOptions[0];
	}

	const startY = e.clientY;
	const startValue = Number(number);
	const handleMouseMove = (e: MouseEvent) => {
		const diff = startY - e.clientY;
		let newValue = startValue + diff;
		if (isNumber(props.minValue) && newValue < props.minValue) {
			newValue = props.minValue;
		}
		if (isNumber(props.maxValue) && newValue > props.maxValue) {
			newValue = props.maxValue;
		}
		handleChange(newValue + "" + unit);
	};
	const handleMouseUp = () => {
		window.removeEventListener("mousemove", handleMouseMove);
	};
	window.addEventListener("mousemove", handleMouseMove);
	window.addEventListener("mouseup", handleMouseUp, { once: true });
};

const clearValue = () => emit("update:modelValue", null);
</script>
