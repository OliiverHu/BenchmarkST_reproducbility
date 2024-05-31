SpatialPCA tutorial
============

1. Dependencies

.. code-block:: r

    library(SpatialPCA)
    library(Seurat)
    library(ggplot2)
    library(Matrix)

2. Data loading: DLPFC

.. code-block:: r

    sample.name <- "151673"
    cluster.number <- 7
    dir.input <- file.path('/data/maiziezhou_lab/Datasets/ST_datasets/DLPFC12/', sample.name)
    dir.output <- file.path('/data/maiziezhou_lab/manfeifei/Projects/Benchmark/R/SpatialPCA/output/', sample.name, '/')
    meta.input <- file.path('/data/maiziezhou_lab/Datasets/ST_datasets/DLPFC12/', sample.name, 'gt')
    layer.input <- file.path('/data/maiziezhou_lab/Datasets/ST_datasets/DLPFC12/', sample.name, 'gt/layered')

    if(!dir.exists(file.path(dir.output))){
        dir.create(file.path(dir.output), recursive = TRUE)
    }

    filename <- paste0(sample.name, "_filtered_feature_bc_matrix.h5")
    sp_data <- Load10X_Spatial(dir.input, filename = filename)

    #df_meta <- read.table(file.path(meta.input, 'metadata.tsv'))
    df_meta <- read.table(file.path(meta.input, 'tissue_positions_list_GTs.txt'))


    original_row_names <- row.names(df_meta) 
    split_data <- strsplit(df_meta$V1, split = ",")
    df_meta <- do.call(rbind, lapply(split_data, function(x) {
        data.frame(V1=x[1], V2=x[2], V3=x[3], V4=x[4], V5=x[5], V6=x[6], V7=x[7])
    }))
    row.names(df_meta) <- df_meta$V1
    df_meta$V3 <- as.numeric(df_meta$V3)
    df_meta$V4 <- as.numeric(df_meta$V4)
    #df_meta_matched <- df_meta[df_meta$V1 %in% row.names(sp_data@meta.data),]
    # Set the row names of df_meta_matched to be V1
    # Identify the cells that are in both sp_data and df_meta
    common_cells <- colnames(sp_data[["Spatial"]]) %in% rownames(df_meta)

    # Subset sp_data to keep only these cells
    sp_data <- sp_data[, common_cells]

    # Initialize an empty dataframe to hold the final results
    layer.data <- data.frame()

    if(as.numeric(cluster.number) == 5) {
    for(i in 3:6){
        file.name <- paste0(sample.name, "_L", i, "_barcodes.txt")
        file.path <- file.path(layer.input, file.name)

        data.temp <- read.table(file.path, header = FALSE, stringsAsFactors = FALSE) # assuming the file has no header
        data.temp <- data.frame(barcode = data.temp[,1], layer = paste0("layer", i), row.names = data.temp[,1])

        # Append to the final dataframe
        layer.data <- rbind(layer.data, data.temp)
    }
    } else {
    for(i in 1:6){
        file.name <- paste0(sample.name, "_L", i, "_barcodes.txt")
        file.path <- file.path(layer.input, file.name)

        data.temp <- read.table(file.path, header = FALSE, stringsAsFactors = FALSE) # assuming the file has no header
        data.temp <- data.frame(barcode = data.temp[,1], layer = paste0("layer", i), row.names = data.temp[,1])

        # Append to the final dataframe
        layer.data <- rbind(layer.data, data.temp)
    }
    }


    # For the WM file
    file.name <- paste0(sample.name, "_WM_barcodes.txt")
    file.path <- file.path(layer.input, file.name)

    data.temp <- read.table(file.path, header = FALSE, stringsAsFactors = FALSE) # assuming the file has no header
    data.temp <- data.frame(barcode = data.temp[,1], layer = "WM", row.names = data.temp[,1])

    # Append to the final dataframe
    layer.data <- rbind(layer.data, data.temp)

    sp_data <- AddMetaData(sp_data, 
                            metadata = df_meta['V3'],
                            col.name = 'row')
    sp_data <- AddMetaData(sp_data, 
                            metadata = df_meta['V4'],
                            col.name = 'col')
    sp_data <- AddMetaData(sp_data, 
                            metadata = layer.data['layer'],
                            col.name = 'layer_guess_reordered')
    count <- sp_data@assays$Spatial@counts

    # get coordinates
    #gtlabels <- list(sp_data@meta.data$layer_guess_reordered)
    coord <- data.frame(row=sp_data@meta.data$row, col=sp_data@meta.data$col)
    row.names(coord) <- row.names(sp_data@meta.data)

3. Run SpatialPCA

.. code-block:: r

    LIBD = CreateSpatialPCAObject(counts=count, location=as.matrix(coord), project = "SpatialPCA",gene.type="spatial",sparkversion="spark",numCores_spark=5,gene.number=3000, customGenelist=NULL,min.loctions = 20, min.features=20)
    LIBD = SpatialPCA_buildKernel(LIBD, kerneltype="gaussian", bandwidthtype="SJ",bandwidth.set.by.user=NULL)
    LIBD = SpatialPCA_EstimateLoading(LIBD,fast=FALSE,SpatialPCnum=20) 
    LIBD = SpatialPCA_SpatialPCs(LIBD, fast=FALSE)  
    clusterlabel <- walktrap_clustering(clusternum=as.numeric(cluster.number),latent_dat=LIBD@SpatialPCs,knearest=70 ) 
    # here for all 12 samples in LIBD, we set the same k nearest number in walktrap_clustering to be 70. 
    # for other Visium or ST data, the user can also set k nearest number as round(sqrt(dim(SpatialPCAobject@SpatialPCs)[2])) by default.
    clusterlabel_refine = refine_cluster_10x(clusterlabels=clusterlabel,location=LIBD@location,shape="hexagon")
    if (length(clusterlabel_refine) != length(sp_data@meta.data$layer_guess_reordered)){
        message1 <- paste("Length of calculated cluster is ", length(clusterlabel_refine))
        message2 <- paste("Length of ground truth label is ", length(sp_data@meta.data$layer_guess_reordered))
        write.table(c(message1, message2), file = file.path(dir.output, "error_message.txt"))
    }

4. Calculate the ARI and save the output

.. code-block:: r

    # Get the common row names
    common_rows <- intersect(rownames(sp_data@meta.data), colnames(LIBD@normalized_expr))

    # Filter rows from sp_data@meta.data
    matched_rows <- sp_data@meta.data[common_rows, ]
    matched_rows[["spatial cluster"]] <- clusterlabel_refine
    
    filename <- paste0(sample.name, "_output.csv")
    write.table(matched_rows, file = file.path(dir.output, filename), sep = "\t", qmethod = "double", col.names=NA)

    gtlabels <- sp_data@meta.data$layer_guess_reordered[match(colnames(LIBD@normalized_expr),colnames(count))]
    ari_spatialpca <- mclust::adjustedRandIndex(clusterlabel_refine, gtlabels)
    # Initialize the result dataframe
    result_df <- data.frame(ari_spatialpca = numeric())
    result_df <- rbind(result_df, data.frame(ari_spatialpca = ari_spatialpca))
    
    # Write the result dataframe to a txt file
    write.table(result_df, file = file.path(dir.output, "ari.txt"), sep = "\t", row.names = FALSE, col.names=TRUE)
