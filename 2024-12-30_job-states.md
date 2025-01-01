# Ingestion | State Transitions

```mermaid
%%---
%%title: "Ingestion State Transitions"
%%---

%%{
  init: {
    "logLevel": "debug",
    "look": "classic",
    "securityLevel": "loose",
    "theme": "forest",
    "themeVariables": {
        "wrap": true,
        "fontSize": "35px",
        "edgeLabelBackground" : {
            "opacity" : 0.0
        }
    },
    "flowchart": {
      "titleTopMargin": 0,
      "subGraphTitleMargin": {
          "top" : 8,
          "bottom" : 32
      },
      "diagramPadding": 32,
      "useMaxWidth": false,
      "defaultRenderer": "dagre-wrapper",
      "curve": "cardinal",
      "htmlLabels": true
    }
  }
}%%

flowchart TD

    %% REQUEST %%
    subgraph ingest-service [" "]
        %% STATES %%
        subgraph states ["States"]
        direction LR
            pending["Pending"]
            in-progress["In Progress"]
            done["Done"]
            error["Error"]
        end

        request@{ shape: paper-tape, label: "Ingest&nbsp;Request" }

        %% ASSET %%
        subgraph request-processing ["Request"]
            subgraph request-type-processing [" "]
                parse-request[[Parse Request]]
                asset-single-path@{label: "Single Asset Path", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/target.png", constraint:on, w:16, h:16}
                asset-folder@{label: "Folderlike\nS3&nbsp;Bucket,&nbsp;GCS&nbsp;Folder,&nbsp;...", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/folder.png", constraint:on, w:16, h:16}
                asset-document@{label: "Listlike\nCSV,&nbsp;JSON,&nbsp;paths,&nbsp;...", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/document.png", constraint:on}
                asset-paths@{label: "Asset&nbsp;Path(s)", shape: docs}
            end
            subgraph asset-type-processing [" "]
                determine-asset-type[["Determine Asset Type"]]
                asset-image@{label: "Image", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/image.png", constraint:on}
                asset-video-keyframe@{label: "Video Keyframe", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/kf-video.png", constraint:on}
                asset-audio-keyframe@{label: "Audio Keyframe", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/kf-audio.png", constraint:on}
                asset-video@{label: "Video", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/video.png", constraint:on}
            end
        end

        %% IMAGE %%
        subgraph image-processing [Image]
            compute-visual-emb[["Compute&nbsp;Visual Embeddings"]]
            embeddings-visual@{ shape: docs, label: "Visual&nbsp;Embeddings" }
        end

        %% VIDEO %%
        subgraph video-processing [Video]
            copy-video[["Copy&nbsp;Video&nbsp;to Our&nbsp;Bucket"]]
            video-file@{label: "Video File", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/film.png", constraint:on}

            %% SHOTS %%
            subgraph shots-processing ["Shots"]
                compute-shot-boundaries[["Compute&nbsp;Shot Boundaries"]]
                shots@{ shape: docs, label: "Shot Boundaries" }
                generate-shot-kf[[Generate Keyframes]]
                shot-kfs@{ shape: docs, label: "Video&nbsp;Keyframe Paths" }
            end

            %% AUDIO %%
            subgraph audio-processing ["Audio"]
                extract-audio[["Extract Audio"]]
                audio-file@{ shape: paper-tape, label: "Audio File" }
                audio-file@{label: "Audio File", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/audio.png", constraint:on}

                %%transcribe-segment-audio[[Transcribe & Segment Audio]]
                transcribe-audio[[Transcribe Audio]]
                transcription@{label: "Transcription", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/transcript.png", constraint:on}
                transcription@{ shape: doc, label: "Transcription" }
                segment-audio[["Split&nbsp;Transcription Into Segments"]]
                segments@{ shape: docs, label: "Segments" }

                compute-text-emb[["Compute&nbsp;Text Embeddings"]]
                embeddings-segments@{ shape: docs, label: "Audio&nbsp;Segment\nTranscript&nbsp;Embeddings" }

                generate-audio-kf[[Generate Keyframes]]
                audio-kfs@{ shape: docs, label: "Audio&nbsp;Keyframe Paths" }
            end
        end

        %% EMBEDDING %%
        subgraph embedding-processing [Embedding]
            write-emb[[Write Embeddings]]
            asset-embedding@{label: "Embedding", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/embedding.png", constraint:on}
            db-emb@{ shape: cyl, label: "Vector DB" }
        end

        asset-embedding-image@{label: "Image\nEmbedding", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/embedding-image.png", constraint:on}
        asset-embedding-kf-video@{label: "Video&nbsp;Keyframe\nEmbedding", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/embedding-kf-video.png", constraint:on}
        asset-embedding-kf-audio@{label: "Audio&nbsp;Keyframe\nEmbedding", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/embedding-kf-audio.png", constraint:on}
        asset-embedding-transcript@{label: "Audio&nbsp;Segment&nbsp;Transcript\nEmbedding", img: "https://raw.githubusercontent.com/coactive-jeffrey/notes/refs/heads/main/assets/embedding-transcript.png", constraint:on}

    end

    %% ASSET
    request --> parse-request
    parse-request --> asset-single-path --> asset-paths
    parse-request --> asset-folder --> asset-paths
    parse-request --> asset-document --> asset-paths

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

    states ~~~ request

    classDef Pending fill:#89CFF0,color:black,stroke:black,stroke-width:5px
    classDef PendingSub fill:transparent,color:black,stroke:black,stroke-width:0px
    classDef InProgress fill:orange,font-weight:italic,stroke:black,stroke-width:5px
    classDef InProgressSub fill:orange,font-weight:italic,stroke:black,stroke-width:2px
    classDef Done fill:green,color:#FFFFFF,stroke:black,font-weight:italic,stroke-width:5px
    classDef Error fill:red,color:white,stroke:black,font-weight:bold,stroke-width:5px

    classDef ImgNodeBlackText fill:transparent,color:black,stroke-width:0px
    class asset-single-path ImgNodeBlackText
    class asset-folder ImgNodeBlackText
    class asset-document ImgNodeBlackText
    class asset-image ImgNodeBlackText
    class asset-video-keyframe ImgNodeBlackText
    class asset-audio-keyframe ImgNodeBlackText
    class asset-video ImgNodeBlackText
    class video-file ImgNodeBlackText
    class audio-file ImgNodeBlackText
    class transcription ImgNodeBlackText
    class asset-embedding-transcript ImgNodeBlackText
    class asset-embedding-image ImgNodeBlackText
    class asset-embedding-kf-video ImgNodeBlackText
    class asset-embedding-kf-audio ImgNodeBlackText

    classDef EmbNodeWhiteText fill:transparent,color:#FFFFFF,stroke-width:0px
    class asset-embedding EmbNodeWhiteText

    pending ==> in-progress ==> done ~~~ error
    %%request-processing ~~~ image-processing
    %%request-processing ~~~ video-processing
    %%image-processing ~~~ embedding-processing
    %%video-processing ~~~ embedding-processing


    classDef Legend fill:white,color:black,stroke:black,stroke-width:5px
    class states, Legend
    class pending, Pending
    class in-progress, InProgress
    class done, Done
    class error, Error

    class request-processing, Pending
    class request-type-processing PendingSub
    class asset-type-processing PendingSub
    class image-processing InProgress
    class video-processing InProgress
    class audio-processing InProgressSub
    class shots-processing InProgressSub
    class embedding-processing Done
```

## Mermaid Version
```mermaid
info
```
