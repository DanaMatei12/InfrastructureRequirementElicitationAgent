import sys
import time
import functions_framework
from google.cloud import storage

def upload_file_from_memory(bucket_name, content, destination_blob_name):

    """Uploads a file to the bucket."""

    #bucket_name: the name of the GCS bucket
    #content: the content to upload
    #destination_blob_name: the name of the uploaded file"

    storage_client = storage.Client()
    bucket = storage_client.bucket(bucket_name)
    blob = bucket.blob(destination_blob_name)

    blob.upload_from_string(content)

# [END upload_file_from_memory]


@functions_framework.http
def infra_agent_http(request):
    
    """Processes a request."""

    #request: the JSON containing configuration data

    content = ""
    request_json = request.get_json(silent=True)

    if request_json:
        for attribute in request_json:
            content = content + attribute + ": " + request_json[attribute] + "\n"

        if request_json['Request_type'] == "BQ_dataset":
            bucket_name = "infra_agent_bq_bucket"
            file_prefix = "Bigquery_dataset_"
        elif request_json['Request_type'] == "CS_bucket":
            bucket_name = "infra_agent_cs_bucket"
            file_prefix = "Storage_bucket_"

        file_name = file_prefix + request_json['Operating_company'] + "_" + time.strftime("%Y%m%d_%H%M%S")
        upload_file_from_memory(bucket_name, content, file_name)
        result = f"The configuration data is saved."
    else:
        result = f"The configuration data was not received."

    res = {"webhook_response": {"message": {"text": [result]}}}
    return res
