<template>
    <div class="relative flex flex-col pt-2 pb-0 eve-container" style="height: calc(100vh - 86px);">
        <div class="flex flex-col justify-start flex-1 px-3">
            <slot> </slot>
            <slot name="form"> </slot>
            <slot name="line"> </slot>
            <slot name="table"> </slot>
        </div>
        <div class="sticky bottom-0 pr-3 bg-white border-0 border-t border-solid border-line-gray pagerbox" style="z-index: 9">
            <slot name="pager"> </slot>
        </div>
    </div>
</template>
<script>

</script>

<style lang="scss">
.eve-container {
    .pagerbox:empty {
        display: none !important;
    }
    .el-divider--horizontal {
        margin: 0 0 12px !important;
        background-color: #f0f0f0;
    }
    .el-table .el-table__header-wrapper th, .el-table .el-table__fixed-header-wrapper th{
        font-size: 12px;
        font-style: normal;
        font-weight: 700;
        line-height: 20px;
    }
    .el-table .el-table__header-wrapper th {
        height: 36px;
        color: #666666;
    }
    .el-table .el-table__header-wrapper th, .el-table .el-table__fixed-header-wrapper th {
        font-size: 12px;
        font-weight: 700;
        height: 28px;
    }
    .el-table--mini td, .el-table--mini th {
        padding: 0 0;
        height: 36px;
        color: #333333;
    }
    .el-button+.el-button{
        margin-left: 10px !important;
    }
    .el-table .el-button--text{
        padding: 0 !important;
    }
    .el-pagination{
        width: 100%;
        text-align: right;
        .el-pagination__total{
            float: left;
            padding-left: 12px;
            &:after {
                visibility: hidden;
                display: block;
                font-size: 0;
                content: " ";
                clear: both;
                height: 0;
            }
        }
    }
    .el-table__header-wrapper.fixed ~ .el-table__fixed-right .el-table__fixed-body-wrapper{
        top: 76px !important;
    }
}
</style>
