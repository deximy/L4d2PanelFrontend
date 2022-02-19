<script lang="ts" setup>
import {NUpload, NUploadTrigger, UploadCustomRequestOptions} from "naive-ui/es";
import {NDataTable} from "naive-ui/es";
import {NIcon} from "naive-ui/es";
import {NDropdown, DropdownOption} from "naive-ui/es";

import {h, nextTick, ref, watch} from "vue";

import {GetFolderList, UploadFile} from "../api";
import {UseStore} from "../Store";
import * as p from "path-browserify";

interface RowData
{
    key: string | number,
    file_type: string,
    file_name: string,
}

const store = UseStore();

const columns = [
    {
        key: "file_type",
        fixed: "left",
        width: "0",
        className: "file_type_icon",
        render: (row: RowData) => {
            if (row.file_type === "directory")
            {
                return h(
                    "div",
                    {
                        class: "file_type_icon_wrapper",
                    },
                    h(
                        NIcon,
                        {

                        },
                        {
                            default: () => {
                                return [
                                    h(
                                        "svg",
                                        {
                                            "xmlns": "http://www.w3.org/2000/svg",
                                            "xmlns:xlink": "http://www.w3.org/1999/xlink",
                                            "viewBox": "0 0 24 24",
                                        },
                                        h(
                                            "path",
                                            {
                                                "d": "M10.59 4.59C10.21 4.21 9.7 4 9.17 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-1.41-1.41z",
                                                "fill": "currentColor",
                                            }
                                        )
                                    )
                                ]
                            }
                        }
                    )
                );
            }
            else
            {
                return h(
                    "div",
                    {
                        class: "file_type_icon_wrapper",
                    },
                    h(
                        NIcon,
                        {

                        },
                        {
                            default: () => {
                                return [
                                    h(
                                        "svg",
                                        {
                                            "xmlns": "http://www.w3.org/2000/svg",
                                            "xmlns:xlink": "http://www.w3.org/1999/xlink",
                                            "viewBox": "0 0 24 24",
                                        },
                                        h(
                                            "path",
                                            {
                                                "d": "M6 2c-1.1 0-1.99.9-1.99 2L4 20c0 1.1.89 2 1.99 2H18c1.1 0 2-.9 2-2V8l-6-6H6zm7 7V3.5L18.5 9H13z",
                                                "fill": "currentColor",
                                            }
                                        )
                                    )
                                ]
                            }
                        }
                    )
                );
            }
        }
    },
    {
        title: "文件名",
        key: "file_name",
        className: "file_name",
    }
];
const data = ref<Array<object>>([]);
const row_props = (row: RowData) => {
    return {
        onDblclick: (e: MouseEvent) => {
            if (row.file_type === "directory")
            {
                EnterDirectory(row.file_name);
            }
            else if (row.file_type === "file")
            {
                OpenFile(row.file_name);
            }
            else
            {
                console.error("Unknown file type!");
            }
        },
        onContextmenu: (e: MouseEvent) => {
            e.preventDefault();
            show_dropdown.value = false;
            store.selected_file_type = row.file_type;
            store.selected_file_name = row.file_name;
            x.value = e.clientX;
            y.value = e.clientY;
            nextTick().then(
                () => {
                    show_dropdown.value = true;
                }
            );
        },
    };
};

let current_path = ref<string>("./");
const RefreshDirectory = async (path: string) => {
    let folder_content = await GetFolderList(path);
    if (folder_content === undefined)
    {
        console.log("Empty response!");
        return;
    }

    data.value.length = 0;
    data.value.push(
        {
            key: "directory_" + "..",
            file_type: "directory",
            file_name: "..",
        }
    );
    for (let i of folder_content.directories)
    {
        data.value.push(
            {
                key: "directory_" + i,
                file_type: "directory",
                file_name: i,
            }
        );
    }
    for (let i of folder_content.files)
    {
        data.value.push(
            {
                key: "file_" + i,
                file_type: "file",
                file_name: i,
            }
        );
    }
};
const EnterDirectory = async (path: string) => {
    current_path.value = p.join(current_path.value, path);
    if (current_path.value.startsWith(".."))
    {
        current_path.value = "./";
    }
};
const OpenFile = async (path: string) => {
    console.log("Try to open file: " + path);
};
const Mkdir = async (directory_name: string) => {
    console.log(directory_name);
};
const DeleteDirectory = async (directory_name: string) => {
    console.log("d:" + directory_name);

};
const DeleteFile = async (file_name: string) => {
    console.log("f:" + file_name);
};

watch(current_path, value => {RefreshDirectory(value)});
RefreshDirectory(current_path.value);

const x = ref(0);
const y = ref(0);
const options: DropdownOption[] = [
    {
        label: "刷新",
        key: "refresh",
    },
    {
        label: "新建文件夹",
        key: "mkdir",
    },
    {
        label: "上传文件",
        key: "upload",
    },
    {
        label: () => h("span", { style: { color: "red" } }, "删除"),
        key: "delete",
    }
];
const show_dropdown = ref(false);
const HandleClickOutSideOfDropdown = () => {
    show_dropdown.value = false;
};
const HandleSelect = (BeginUpload: () => void, key: string) => {
    show_dropdown.value = false;

    if (key === "refresh")
    {
        RefreshDirectory(current_path.value);
    }
    else if (key === "mkdir")
    {
        console.log("mkdir");
        Mkdir("");
    }
    else if (key === "upload")
    {
        BeginUpload();
    }
    else if (key === "delete")
    {
        if (store.selected_file_type === "directory")
        {
            DeleteDirectory(store.selected_file_name);
        }
        else if (store.selected_file_type === "file")
        {
            DeleteFile(store.selected_file_name);
        }
        else
        {
            console.error("Unknown file type:" + store.selected_file_type);
        }
    }
    else
    {
        console.error("Unknown key: " + key);
    }
};

const CustomUploadFile = (
    {
        file: file_info,
        action,
        onFinish,
        onError,
    }: UploadCustomRequestOptions
) => {
    console.log(file_info.file);
    console.log(action);
    console.log(onFinish);
    console.log(onError);
    UploadFile(file_info.file as File, current_path.value, onFinish, onError);
};
</script>

<template>
    <div class="file_manager">
<!--        <NButton @click="GetPathFiles">浏览目录</NButton>-->
<!--        <NUpload-->
<!--            action="https://localhost:5001/upload"-->
<!--            :custom-request="UploadFile"-->
<!--        >-->
<!--            <NButton>上传文件</NButton>-->
<!--        </NUpload>-->
        <n-data-table
            class="file_list"
            :bordered="false"
            :columns="columns"
            :data="data"
            :row-props="row_props"
        />
        <n-upload abstract :custom-request="CustomUploadFile">
            <n-upload-trigger abstract #="{handleClick: HandleClick}">
                <n-dropdown
                    placement="bottom-start"
                    trigger="manual"
                    :x="x"
                    :y="y"
                    :options="options"
                    :show="show_dropdown"
                    @clickoutside="HandleClickOutSideOfDropdown"
                    @select="HandleSelect(HandleClick, $event)"
                />
            </n-upload-trigger>
        </n-upload>

    </div>
</template>

<style scoped>
.controller
{
    margin: 20px 2% 60px 2%;
    display: flex;
    align-content: center;
    justify-content: space-around;
}

.file_manager
{
    height: 100%;
}
.file_list
{
    height: 100%;
}
:deep(.file_type_icon)
{
    padding-right: 0;
}
/*:deep(.file_name)*/
/*{*/
/*    padding-left: 0;*/
/*}*/
:deep(.file_type_icon_wrapper)
{
    display: flex;
}
</style>
