```C++
#include "google/protobuf/util/json_util.h"
#include <fstream>

void Process() {
  std::ofstream json_file("output.json");
  if (!json_file.is_open()) {
    return;
  }

  for (const auto &log : logs_) {
    rec::ProductResponse response;
    response.ParseFromString(log.second.resp_prerank());
    std::string str_response;
    google::protobuf::util::MessageToJsonString(response, &str_response);

    json_file << str_response;
  }
  json_file.close();
}
```

```C++
#include <csv.h>
#include <glog/logging.h>
#include <fstream>
#include <string>
#include <unordered_map>
#include "ai_platform/idls/tess/legacy/online.pb.h"
#include "context.pb.h"
#include "google/protobuf/util/json_util.h"
#include "oxygen/util/base64.h"
#include "tesseract/merger3/common/tess2_utils.h"
#include "tesseract/merger_idls/caller/common/common.pb.h"
#include "tesseract/merger_idls/caller/search/search.pb.h"
#include "tesseract/merger_idls/internal/kafka/kafka.pb.h"
#include "tesseract/merger_idls/legacy/search/search.pb.h"
#include "userRT.pb.h"

namespace merger {

class Reader : public io::CSVReader<3> {
 private:
  std::unordered_map<std::string, kafka::MergerLog> logs_;

 public:
  explicit Reader(const std::string& csv) : io::CSVReader<3>(csv) {
    std::string decode;
    std::string log_id, scene, content;
    read_header(io::ignore_extra_column, "log_id", "scene", "content");
    while (read_row(log_id, scene, content) && oxygen::Base64Decode(content, &decode)) {
      logs_[log_id].ParseFromString(decode);
    }
  }

  void Process() {
    std::ofstream json_file("0929data.json");
    if (!json_file.is_open()) {
      return;
    }

    for (const auto& log : logs_) {
      search::SearchRequest request;
      request.ParseFromString(log.second.req());
      std::string str_response;
      google::protobuf::util::MessageToJsonString(request, &str_response);

      json_file << str_response;
    }
    json_file.close();
  }
};

int main(int, char**) {
  Reader reader("0929_1300search.csv");
  reader.Process();

  return 0;
}

}  // namespace merger

int main(int argc, char** argv) { return merger::main(argc, argv); }


```
