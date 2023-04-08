<%
// ------------------------------------------------------------------------
// File:    topics.pas.md
// Author:  (c) 2023 by Jens Kallup - paule32 - kallup non-profit Software
// License: MIT - Code of Conduct.
//
// This HelpNDoc.exe file replace the markdown link: [link](<file.md>) to
// Borland TurboVision help link: {Link:_file_md_}.
// You can compile the resulting text (file) with the tvhc.exe compiler
// to get a binary help file, that you can use in your TV-Pascal/C++
// Application Project's.
// ------------------------------------------------------------------------
var topicContent: TStringList;
var i,j: Integer;
var s:    String;

// hard coded output file path for the help project files:
const outpath = 'E:\Projekte\HelpNDocTemplateTV\output';

// ------------------------------------------------------------------------
// this is the main "replace" function ...
// ------------------------------------------------------------------------
function replaceLinkTopic(topicText: String): String;
var i,j,k,l: Integer;
var s,t,u,v,w: String;
begin
  result := topicText;

  s := topicText;
  i := Pos('[',s);
  j := PosEx(']',s,i);

  k := PosEx('(',s,j);
  l := PosEx(')',s,k);

  u := Copy(s,k+1,l-k-1);
  t := Copy(s,i+1,j-i-1);

  w := '[' + t + '](' + u + ')';

  u := StringReplace(u,'<','_',[rfReplaceAll]);
  u := StringReplace(u,'>','_',[rfReplaceAll]);
  u := StringReplace(u,'.','_',[rfReplaceAll]);

  v := '{' + t + ':' + u + '}';

  s := StringReplace(s,w,v,[rfReplaceAll]);
  result := s;
end;

// ------------------------------------------------------------------------
// pascal script start entry ...
// ------------------------------------------------------------------------
try    // finally
  try  // except
    topicContent := TStringList.Create;
    topicContent.writeBOM := false;

    HndGeneratorInfo.BOMOutput := False;
    HndGeneratorInfo.ForceOutputEncoding := True;

    // Get a list of generated topics
    var aTopicList := HndTopicsEx.GetTopicListGenerated(False, False);

    // Default topic
    var sDefaultTopicId := HndProjects.GetProjectDefaultTopic();

    // Each individual topics...
    for var nCurTopic := 0 to length(aTopicList) - 1 do
    begin
      // Notify about the topic being generated
      HndGeneratorInfo.CurrentTopic := aTopicList[nCurTopic].id;

      // Topic kind
      if (aTopicList[nCurTopic].Kind = 1) then continue;  // Empty topic: do not generate anything

      // Setup the file name
      HndGeneratorInfo.CurrentFile := aTopicList[nCurTopic].HelpId + '.txt';

      // content file ...
      topicContent.Clear;
      topicContent.Text := HndTopics.GetTopicContentAsMarkdown(HndGeneratorInfo.CurrentTopic);
      s := HndTopics.GetTopicCaption(aTopicList[nCurTopic].Id) + '_md_';

      s := StringReplace(s,'ü','u',[rfReplaceAll]); s := StringReplace(s,'Ü','U',[rfReplaceAll]);
      s := StringReplace(s,'ä','a',[rfReplaceAll]); s := StringReplace(s,'Ä','A',[rfReplaceAll]);
      s := StringReplace(s,'ö','o',[rfReplaceAll]); s := StringReplace(s,'Ö','o',[rfReplaceAll]);

      s := StringReplace(s,#13,'' ,[rfReplaceAll]);
      s := '.topic _' + s + #10;

      for i := 0 to topicContent.Count - 3 do
      s := s + topicContent[i] + #10;
      s := replaceLinkTopic(s);

      topicContent.Text := s;
      topicContent.SaveToFile(outpath + '\' + HndGeneratorInfo.CurrentFile);
    end;
  except
    on E: Exception do
    begin
      ShowMessage('Error:' + #1013 +
      E.Message);
      exit;
    end;
  end;
finally
  topicContent.Clear;
  topicContent.Free;
end;
%>