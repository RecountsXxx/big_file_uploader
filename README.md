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
```
