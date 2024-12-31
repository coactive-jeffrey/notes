# Ingestion | State Transitions

```mermaid
---
title: "Ingestion | State Transitions"
---
%%{
  init: {
    "logLevel": "debug",
    "look": "classic",
    "fontFamily": "monospace",
    "securityLevel": "loose",
    "theme": "forest",
    "themeVariables": {
        "edgeLabelBackground" : {
            "opacity" : 0.0
        }
    },
    "flowchart": {
      "useMaxWidth": true,
      "defaultRenderer": "elk",
      "curve": "curve",
      "htmlLabels": true
    },
    "elk": {
        "mergeEdges" : true,
        "nodePlacementStrategy" : "BRANDES_KOEPF"
    }
  }
}%%

flowchart TD



    %% REQUEST %%
    subgraph ingest-service [Ingest Service]
        request@{ shape: paper-tape, label: "Ingest Request" }

        %% ASSET %%
        subgraph request-processing [Request Processing]
            subgraph request-type-processing [Obtain Paths]
                parse-request[[Parse Request]]
                asset-paths@{label: "Asset Path(s)", shape: docs}
                asset-folder@{label: "Folderlike\nS3 Bucket, GCS Folder, ...", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/folder.png", constrain:on}
                asset-document@{label: "Listlike\nCSV, JSON, paths, ...", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/document.png", constrain:on}
            end
            subgraph asset-type-processing [Request Processing]
                determine-asset-type[[Determine Asset Type]]
                asset-image@{label: "Image", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/image.png", constrain:on}
                asset-video-keyframe@{label: "Video Keyframe", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/kf-video.png", constrain:on}
                asset-audio-keyframe@{label: "Audio Keyframe", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/kf-audio.png", constrain:on}
                asset-video@{label: "Video", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/video.png", constrain:on}
            end
        end

        %% IMAGE %%
        subgraph image-processing [Image]
            compute-visual-emb[[Compute Visual Embeddings]]
            embeddings-visual@{ shape: docs, label: "Visual Embeddings" }
        end

        %% VIDEO %%
        subgraph video-processing [Video]
            copy-video[[Copy Video]]
            video-file@{ shape: paper-tape, label: "Video File" }

            %% SHOTS %%
            subgraph shots-processing [Shots Processing]
                compute-shot-boundaries[[Compute Shot Boundaries]]
                shots@{ shape: docs, label: "Shot Boundaries" }
                generate-shot-kf[[Generate Keyframes]]
                shot-kfs@{ shape: docs, label: "Video Keyframe\nPaths" }
            end

            %% AUDIO %%
            subgraph audio-processing [Audio Processing]
                extract-audio[[Extract Audio]]
                audio-file@{ shape: paper-tape, label: "Audio File" }

                %%transcribe-segment-audio[[Transcribe & Segment Audio]]
                transcribe-audio[[Transcribe Audio]]
                transcription@{label: "Transcription", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/transcript.png", constrain:on}
                transcription@{ shape: doc, label: "Transcription" }
                segment-audio[[Segment Transcription]]
                segments@{ shape: docs, label: "Segments" }

                compute-text-emb[[Compute Text Embeddings]]
                embeddings-segments@{ shape: docs, label: "Audio Segment\nTranscript Embeddings" }

                generate-audio-kf[[Generate Keyframes]]
                audio-kfs@{ shape: docs, label: "Audio Keyframe\nPaths" }
            end
        end

        %% EMBEDDING %%
        subgraph embedding-processing [Embedding]
            write-emb[[Write Embeddings]]
            asset-embedding@{label: "Embedding", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/embedding.png", constrain:on}
            db-emb@{ shape: cyl, label: "Vector DB" }
        end

        %% STATES %%
        subgraph states ["States"]
            pending["Pending"]
            in-progress["In Progress"]
            done["Done"]
            error["Error"]
        end
            asset-embedding-image@{label: "Image\nEmbedding", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/embedding-image.png", constrain:on}
            asset-embedding-kf-video@{label: "Video Keyframe\nEmbedding", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/embedding-kf-video.png", constrain:on}
            asset-embedding-kf-audio@{label: "Audio Keyframe\nEmbedding", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/embedding-kf-audio.png", constrain:on}
            asset-embedding-transcript@{label: "Audio Segment\nTranscript Embedding", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/embedding-transcript.png", constrain:on}

    end

    %% ASSET
    request --> parse-request
    parse-request --> asset-folder --> asset-paths
    parse-request --> asset-document --> asset-paths
    parse-request --> asset-paths

    asset-paths --> determine-asset-type

    determine-asset-type
        --> asset-image
    determine-asset-type
        --> asset-video-keyframe
    determine-asset-type
        --> asset-audio-keyframe
    determine-asset-type
        --> asset-video

    asset-image --> compute-visual-emb
    asset-video-keyframe --> compute-visual-emb
    asset-audio-keyframe --> compute-visual-emb
    asset-video --> copy-video

    %% IMAGE
        compute-visual-emb --> embeddings-visual

    %% VIDEO
    copy-video
        --> video-file
        --> compute-shot-boundaries --> shots
        --> generate-shot-kf --> shot-kfs

    %% AUDIO
    video-file
        --> extract-audio --> audio-file
        %%--> transcribe-segment-audio --> segments
        --> transcribe-audio --> transcription
        --> segment-audio --> segments
        --> compute-text-emb --> embeddings-segments
    segments
        --> generate-audio-kf --> audio-kfs

    %% KEYFRAMES
    shot-kfs --> determine-asset-type
    audio-kfs --> determine-asset-type

    %% EMBEDDINGS
    embeddings-visual --> asset-embedding-image --> write-emb
    embeddings-visual --> asset-embedding-kf-video --> write-emb
    embeddings-visual --> asset-embedding-kf-audio --> write-emb
    embeddings-segments --> asset-embedding-transcript --> write-emb
    write-emb --> asset-embedding --> db-emb

    classDef Pending fill:#89CFF0,color:black,stroke:black,stroke-width:5px
    classDef InProgress fill:orange,font-weight:italic,stroke:black,stroke-width:5px
    classDef Done fill:green,color:white,stroke:black,font-weight:bold,stroke-width:5px
    classDef Error fill:red,color:white,stroke:black,font-weight:bold,stroke-width:5px

    classDef ImgNodeBlackText fill:transparent,color:black,stroke-width:0px
    class asset-folder ImgNodeBlackText
    class asset-document ImgNodeBlackText
    class asset-image ImgNodeBlackText
    class asset-video-keyframe ImgNodeBlackText
    class asset-audio-keyframe ImgNodeBlackText
    class asset-video ImgNodeBlackText
    class transcription ImgNodeBlackText
    class asset-embedding-transcript ImgNodeBlackText
    class asset-embedding-image ImgNodeBlackText
    class asset-embedding-kf-video ImgNodeBlackText
    class asset-embedding-kf-audio ImgNodeBlackText

    classDef EmbNodeWhiteText fill:transparent,color:white,stroke-width:0px
    class asset-embedding EmbNodeWhiteText

    pending ==> in-progress ==> done ~~~ error


    classDef Legend fill:white,color:black,stroke:black,stroke-width:5px
    class states, Legend
    class pending, Pending
    class in-progress, InProgress
    class done, Done
    class error, Error

    class request-processing, Pending
    class request-type-processing Pending
    class asset-type-processing Pending
    class image-processing InProgress
    class video-processing InProgress
    class audio-processing InProgress
    class shots-processing InProgress
    class embedding-processing Done
```
```mermaid
info
```
