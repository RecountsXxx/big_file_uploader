## Laravel controller code and Blade template

```php
class UploadController extends Controller
{
    public function upload(Request $request)
    {
        $chunk = $request->file('file');
        $currentChunk = $request->input('currentChunk');
        $totalChunks = $request->input('totalChunks');
        $fileName = $request->input('fileName');

        $filePath = 'chunks/' . $fileName . '_' . $currentChunk;
        Storage::put($filePath, file_get_contents($chunk));

        if ($currentChunk == $totalChunks - 1) {
            $chunks = [];
            for ($i = 0; $i < $totalChunks; $i++) {
                $chunks[] = 'chunks/' . $fileName . '_' . $i;
            }
            $content = '';
            foreach ($chunks as $chunkFile) {
                $content .= Storage::get($chunkFile);
                Storage::delete($chunkFile);
            }
            $finalPath = 'uploads/' . $fileName;
            Storage::put($finalPath, $content);

            return response()->json(['success' => true, 'path' => $finalPath]);
        }

        return response()->json(['success' => true]);
    }
}
## Laravel controller code and Blade template
<form id="uploadForm" enctype="multipart/form-data">
    @csrf
    <input type="file" id="myfile" name="myfile">
    <button type="button" id="submitBtn">Отправить</button>
</form>

<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
<script>
    $(document).ready(function() {
        $('#submitBtn').click(function(e) {
            e.preventDefault();
            var fileInput = $('#myfile')[0].files[0];
            var chunkSize = 2000000;
            var totalChunks = Math.ceil(fileInput.size / chunkSize);
            var currentChunk = 0;
            var start = 0;
            var end = Math.min(chunkSize, fileInput.size);

            function uploadChunk() {
                var formData = new FormData();
                var chunk = fileInput.slice(start, end);
                formData.append('file', chunk);
                formData.append('fileName',fileInput.name);
                formData.append('totalChunks', totalChunks);
                formData.append('currentChunk', currentChunk);
                formData.append('_token', '{{ csrf_token() }}');

                $.ajax({
                    type: 'POST',
                    url: "{{ route('upload.chunk') }}",
                    data: formData,
                    processData: false,
                    contentType: false,
                    success: function(response) {
                        console.log(response);
                        currentChunk++;
                        start = end;
                        end = Math.min(start + chunkSize, fileInput.size);
                        if (currentChunk < totalChunks) {
                            uploadChunk();
                        } else {
                            console.log('File uploaded successfully');
                        }
                    },
                    error: function(xhr, status, error) {
                        console.error(xhr.responseText);
                    }
                });
            }

            uploadChunk();
        });
    });
</script>
