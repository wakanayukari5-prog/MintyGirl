# MintyGirl

# MintyGirlusing System.IO;
using UnityEngine;
using UnityEditor;
using UniVRM10;
using VRMShaders;

#if UNITY_EDITOR
public static class VRMExporterEditor
{
    [MenuItem("VRM1/Export MintyGirl")]
    public static void ExportMenu()
    {
        // シーン内に "MintyGirl" があればそれを優先、なければ選択中のオブジェクトを使用
        GameObject target = GameObject.Find("MintyGirl") ?? Selection.activeGameObject;
        if (target == null)
        {
            EditorUtility.DisplayDialog("Export VRM", "Target model not found. Place an object named 'MintyGirl' in the scene or select the target GameObject.", "OK");
            return;
        }

        string defaultFileName = "MintyMagicGirl.vrm";
        string path = EditorUtility.SaveFilePanel("Export VRM (VRM 1.0)", Application.dataPath, defaultFileName, "vrm");
        if (string.IsNullOrEmpty(path))
        {
            Debug.Log("Export cancelled.");
            return;
        }

        ExportAsync(target, path);
    }

    static async void ExportAsync(GameObject source, string path)
    {
        // 作業用コピーを作成してシーンを汚さない
        var copy = Object.Instantiate(source);
        copy.name = source.name + "_ExportCopy";

        try
        {
            // 不要なエディタ専用オブジェクト/コンポーネントの除去（必要に応じて拡張）
            RemoveEditorOnlyComponents(copy);

            // メタ情報を作成
            var meta = ScriptableObject.CreateInstance<VRM10Meta>();
            meta.Name = "Minty Magic Girl（仮）";
            meta.Version = "1.0";
            meta.Authors = new[] { "wakana yukari" };
            meta.CopyrightInformation = "";
            meta.ContactInformation = "Vrmxporter";
            meta.Thumbnail = null; // 任意：Texture2D を設定すればサムネが埋まります

            // エクスポート設定（必要に応じてフィールドを埋める）
            var settings = new VRM10ExportSettings
            {
                // 例: enable/disable features がある場合はここで設定
            };

            // ----------------------------------------------------------------
            // UniVRM のバージョンによっては同期 API (Export) か非同期 API (ExportAsync) があります。
            // 下は「同期 API が byte[] を返す」場合の呼び出し例です（place-holder）。
            // bytes = VRM10Exporter.Export(copy, settings, meta);
            //
            // もし UniVRM が非同期 API を提供しているなら、例えば:
            // bytes = await VRM10Exporter.ExportAsync(copy, settings, meta);
            //
            // ご使用の UniVRM バージョンに合わせて適宜置き換えてください。
            // ----------------------------------------------------------------

            byte[] bytes = null;

            // ----------------- PLACEHOLDER: Replace with actual API call -----------------
            // Try synchronous call first (commonly used in many UniVRM examples)
            bytes = VRM10Exporter.Export(copy, settings, meta);
            // --------------------------------------------------------------------------------

            if (bytes == null)
            {
                throw new System.Exception("Export returned null bytes. Check UniVRM API compatibility.");
            }

            File.WriteAllBytes(path, bytes);

            if (path.StartsWith(Application.dataPath))
            {
                AssetDatabase.Refresh();
            }

            EditorUtility.DisplayDialog("Export VRM", $"Exported VRM to:\n{path}", "OK");
            EditorUtility.RevealInFinder(path);
            Debug.Log($"Exported VRM (VRM 1.0) to {path}");
        }
        catch (System.Exception ex)
        {
            Debug.LogError($"VRM export failed: {ex}");
            EditorUtility.DisplayDialog("Export Failed", ex.Message, "OK");
        }
        finally
        {
            // 後片付け
            Object.DestroyImmediate(copy);
        }
    }

    static void RemoveEditorOnlyComponents(GameObject root)
    {
        // EditorOnly タグつきオブジェクトの削除（必要に応じて除外条件を追加）
        foreach (var go in root.GetComponentsInChildren<Transform>(true))
        {
            if (go.gameObject.CompareTag("EditorOnly"))
            {
                Object.DestroyImmediate(go.gameObject);
            }
        }

        // さらに特定のエディタ専用コンポーネントを除去したい場合はここに追加
    }
}
#endif
