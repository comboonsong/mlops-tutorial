```mermaid
flowchart TB
    %% Main starting point
    start([Start DICOM Completeness Check]) --> py1
    
    %% Python process
    subgraph Python["Python Script: check_completeness.py"]
        py1["Initialize Logger"] --> py2["Check DICOM Files Existence"]
        py2 --> pyDecision{"DICOMDIR and<br>DICOM Exist?"}
        pyDecision -->|No| pyFail["Log Error & Fail"]
        pyDecision -->|Yes| py3["Call MATLAB<br>(via run_command)"]
        py3 --> py4["Sync JSON from MATLAB"]
        py4 --> py5["Complete Process"]
    end
    
    %% MATLAB process
    subgraph MATLAB["MATLAB Function: check_completeness.m"]
        m1["Load Configuration<br>(check_completeness_config.m)"] --> m2["Initialize Variables"]
        m2 --> m3["Load DICOMDIR Information"]
        m3 --> m4["Find Series Paths"]
        
        %% Handling large number of paths
        m4 --> mPathCount{"Many DICOM<br>Paths (>500)?"}
        mPathCount -->|Yes| mSetSpeed["Set Skip Interval<br>(dicom_read_speed=30)"]
        mPathCount -->|No| mProcess["Process Each Path"]
        mSetSpeed --> mProcess
        
        mProcess --> mFileCheck{"File Exists?"}
        mFileCheck -->|No| mTryDCM["Try with .dcm Extension"]
        mTryDCM --> mDCMCheck{"DCM File<br>Exists?"}
        mDCMCheck -->|No| mFileError["Error: DICOM File Not Found"]
        mFileCheck -->|Yes| mExtract["Extract DICOM Info"]
        mDCMCheck -->|Yes| mExtract
        
        mExtract --> m5["Build Modalities List"]
        m5 --> m6["Check Required Modalities"]
        
        m6 --> mJsonCheck{"JSON File<br>Exists?"}
        mJsonCheck -->|No| mJsonError["Error: JSON Not Found"]
        mJsonCheck -->|Yes| m7["Load JSON"]
        
        m7 --> m8["Update JSON Structure"]
        m8 --> m9["Check Completeness"]
        m9 --> m10["Check if All Required<br>Modalities Found"]
        m10 --> mWriteJSON["Write Updated JSON"]
        
        %% JSON Write error handling
        mWriteJSON --> mTryCatch{"Write<br>Successful?"}
        mTryCatch -->|No| mWriteError["Error: JSON Write Failed"]
        mTryCatch -->|Yes| mSuccess["Return Results"]
    end
    
    %% Config details
    subgraph Config["Configuration: check_completeness_config.m"]
        c1["Required Modalities:<br>
        - fMRI Working memory<br>
        - Brain T2* 0.88/1.1<br>
        - DTI_medium_iso<br>
        - SAG T1W_3D_TFE<br>
        - 3D_Brain_VIEW_T2_Tra"]
        
        c2["Path Constants:<br>
        - ROOT_PATH = data/staging_data<br>
        - DICOMDIR_BASE_PATH = dicom/DICOMDIR<br>
        - DIRECTORY_RECORD_TYPE = IMAGE"]
    end
    
    %% Data paths
    subgraph DataPaths["File Paths"]
        d1["Subject Directory:<br>
        data/staging_data/subject_id/"]
        
        d2["DICOM Directory:<br>
        data/staging_data/subject_id/dicom/"]
        
        d3["JSON Log:<br>
        data/staging_logs/subject_id/task_status.json"]
    end
    
    %% Connections between components
    py3 ==> m1
    m1 -.-> c1
    m1 -.-> c2
    mSuccess ==> py4
    
    %% Data connections
    py1 -.-> d3
    m3 -.-> d1
    m3 -.-> d2
    mWriteJSON -.-> d3
    
    %% Error paths
    mFileError -.-> pyFail
    mJsonError -.-> pyFail
    mWriteError -.-> pyFail
    
    %% Output paths
    py5 --> output["Complete JSON with<br>Modality Status &<br>Completeness Check"]
    
    pyFail --> fail["Failed Process"]
    
    %% Styling
    classDef python fill:#4B8BBE,stroke:#306998,color:white;
    classDef matlab fill:#BB0000,stroke:#8B0000,color:white;
    classDef config fill:#FFC300,stroke:#DAA520,color:black;
    classDef data fill:#82B366,stroke:#5D8444,color:white;
    classDef endpoint fill:#72A0C1,stroke:#4682B4,color:white;
    classDef error fill:#FF6347,stroke:#8B0000,color:white;
    
    class Python python;
    class MATLAB matlab;
    class Config config;
    class DataPaths data;
    class start,output,fail endpoint;
    class mFileError,mJsonError,mWriteError error;
```s
