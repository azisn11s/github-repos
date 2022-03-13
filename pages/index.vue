<template>
	<div>
		<!-- As a link -->
		<nav class="navbar navbar-light" style="background-color: #e3f2fd">
			<div class="container-fluid">
				<a class="navbar-brand" href="#">Github Repositories</a>
			</div>
		</nav>

		<div class="container mt-5">
      <div class="row">
        <div class="col-12">
          <h3>Show {{ githubUsername }} Github Repositories</h3>
          <div class="card">
            <div class="card-header">
              <h3 class="card-title">List</h3>
              <div class="card-tools">
                <div class="input-group input-group-sm" style="max-width: 200px;">
                  <input type="text" v-model="searchFilter" @keyup="()=> { this._.delay(()=> this.getDataFromApi(), 1000) }" class="form-control float-right" placeholder="Search">

                  <div class="input-group-append">
                  <button type="submit" class="btn btn-default" @click="()=> { this.getDataFromApi();}">
                    <i class="fas fa-search"></i>
                  </button>
                  </div>
                </div>
              </div>
            </div>
            <div class="card-body">
              <vue-element-loading :active="isLoading" :is-full-screen="false" style="z-index: 1"/>
              <ve-table :columns="columns" :table-data="tableData" id="table-container" :sort-option="sortOption" />
              <div class="table-pagination">
                              <!-- 'sizer', 'jumper' -->
                <ve-pagination
                  :total="totalCount"
                  :page-index="pageIndex"
                  :page-size="pageSize"
                  @on-page-number-change="pageNumberChange"
                  @on-page-size-change="pageSizeChange"
                  :layout="['prev', 'pager', 'next']"
                />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
	</div>
</template>

<script>
export default {
	name: "IndexPage",

  async asyncData({$axios, query}){
    let githubUsername= 'defunkt';

    if (query.username && query.username.length) {
      githubUsername = query.username;
    }

    // const repositories = await $axios.get(`https://api.github.com/users/defunkt/repos`, {
    //   query: {
    //     page: 2,
    //     per_page: 3
    //   }
    // })

    return {
      // repositories,
      githubUsername
    }
  },

  data(){
    return {
			isLoading: false,
      loadingInstance: null,
			columns: [
				{
					field: "",
					key: "a",
					title: "No.",
					width: 50,
					align: "center",
					renderBodyCell: ({ row, column, rowIndex }, h) => {
						// return ++rowIndex;
            return (this.pageIndex - 1) * this.pageSize + 1 + rowIndex;
					}					
				},
				{ field: "name", key: "b", title: "Repo's Name", align: "left", sortBy: "" },
				{ 
					field: "visibility", key: "c", title: "Visibility", align: "center", sortBy: "",
				},
				
				{
					field: "url",
					key: "d",
					title: "URL",
					align: "left",
          sortBy: "",
				},
				{
					field: "language",
					key: "e",
					title: "Language",
					align: "left",
					sortBy: ""
				},
				{
					field: "watchers_count",
					key: "f",
					title: "Watcher",
					align: "center",
					sortBy: ""
				},
				{
					field: "created_at",
					key: "g",
					title: "Created At",
					align: "center",
          sortBy: "",
				},
        {
					field: "updated_at",
					key: "h",
					title: "Updated At",
					align: "center",
          sortBy: "",
				},
			],
			// page index
			pageIndex: 1,
			// page size
			pageSize: 10,
			totalCount: 0,
			tableData: [],
      searchFilter: "",
			sortParams: "",
			sortDirection: "",
			sortOption: {
				// sort always
				sortAlways: true,
				sortChange: (params) => {
					console.log("sortChange::", params);
          Object.keys(params).forEach((item, idx)=> {
            console.log('ITEM', item);
            console.log('VALUE', params[item]);
            

            if (item && params[item]) {
              this.sortParams = item;
              this.sortDirection = params[item];

              this.$nextTick(()=>{
                this.getDataFromApi();
              });
            }
          })
					// if (Object.keys(params).length) {
					// 	this.sortParams = params;
						
					// }
          
				},
			},
		};
  },

  methods: {
    /**
		 * Required methods
		 * 
		 */
		// page number change
		pageNumberChange(pageIndex) {
			this.pageIndex = pageIndex;
            this.getDataFromApi();
		},

		// page size change
		pageSizeChange(pageSize) {
			this.pageIndex = 1;
			this.pageSize = pageSize;
            this.getDataFromApi();
		},

		// Get table data from API
		async getDataFromApi() {
            // this.loadingInstance.show();
			this.isLoading = true;
			try {
				const { data } = await this.$axios.get(`https://api.github.com/users/${this.githubUsername}/repos`, {
					params: {
						page: this.pageIndex,
						per_page: this.pageSize,
						search: this.searchFilter,
						sort: this.sortParams,
            direction: this.sortDirection
					},
				});

				this.totalCount = 1000; // data.length;
				this.tableData = data;
				this.isLoading = false;


			} catch (error) {
				this.isLoading = false;
				console.log("ERROR!!!!!!!!!!!", error);
				this.$swal.fire('Fetch data failed!', '', 'danger');

			}
		},
  },

  /**
	 * Lifecycle
	 * 
	 */
	created() {
		this.getDataFromApi();
	},

};
</script>
