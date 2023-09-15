{ 
  Inline variables and consts support for FPC.
  Implemented for personal usage, feel free to modify/fix/include everywhere you want.
  Target was to implement as close to Delphi implementation as possible, but there
    are few minor inconsistances in type detection (see TestTypeDetect) and
    in order of init/fin managed types (see ClassicVarOrder/InlineVarOrder1/InlineVarOrder2).
  Known issue : inline variable for anon proc inside generic class/method aren't working (see TClass1<T>.D1).
  
  Usage for vars (see more in tests):
    var x : Integer;
    var x,y,z : String;
    var x : Integer := 1;
    var x := 1;
    var Dict := TDictionary<String, TPair<Integer, Double>>.Create;
    var pr1 := function : Integer begin Result := 3; end;
    for var ind := 1 to 10 do
    for var ind in Dict do
    
  Usage for const:
    const x : Integer = 1;
    const x = 1;
    const Dict = TDictionary<String, TPair<Integer, Double>>.Create;
    
  Note : multiple vars with same name, e.g.
    if true then
    begin
      var x := 1;  //"x"
    end else
    begin
      begin
        var x := 'test';  //"x___1"
      end;
      begin
        var x := 3.4; //"x___2"
      end;
    end;
    
    will work fine (as expected) but debugger ide (lazarus) will always show first variable (on mouse hover or inside watches),
    although it's possible to access others inside watches by adding suffix ( ___1   ___2  etc);
}

{$Mode DELPHIUNICODE}
{$ModeSwitch implicitfunctionspecialization}
{$ModeSwitch functionreferences}
{$ModeSwitch anonymousfunctions}
//{$Define DelphiRtl}

library TestInlineVariables;

{$IfDef DelphiRtl}
  uses System.Generics.Collections;
{$EndIf}

type
  TManagedRecord = record
    class operator Initialize(var Dest: TManagedRecord);
    class operator Finalize(var Dest: TManagedRecord);
    procedure Ping;
  end;
  
class operator TManagedRecord.Initialize(var Dest: TManagedRecord);
begin
  Writeln('TManagedRecord.Initialize');
end;

class operator TManagedRecord.Finalize(var Dest: TManagedRecord);
begin
  Writeln('TManagedRecord.Finalize');
end;

procedure TManagedRecord.Ping;
begin
  Writeln('TManagedRecord.Ping');
end;



//////////////////////////////////////////////////////
procedure ClassicVarOrder();
var
  rec : TManagedRecord;
begin
  Writeln('ClassicVarOrder.begin');
  rec.Ping;
  Writeln('ClassicVarOrder.end');  
end;
  
procedure InlineVarOrder1(); //Inconsistet with Delphi
begin
  Writeln('ClassicVarOrder.begin');
  var rec : TManagedRecord;
  rec.Ping;
  Writeln('ClassicVarOrder.end');  
end;

procedure InlineVarOrder2(); //Inconsistet with Delphi
begin
  Writeln('ClassicVarOrder.begin');
  begin
    var rec : TManagedRecord;
    rec.Ping;
  end;
  Writeln('ClassicVarOrder.end');  
end;

procedure DuplicateVarValid1();
begin
  var X1 : Integer;
  X1 := 1;
  begin 
    var X2 : Integer;
    X2 := X1 + 1;
    Writeln(X2);
  end;
  begin 
    var X2 : Integer;
    X2 := X1 + 2;
    Writeln(X2);
  end;
end;

var 
  DuplicateVarValid_X1 : Integer;
procedure DuplicateVarValid2();
begin
  DuplicateVarValid_X1 := 2;
  Writeln(DuplicateVarValid_X1);
  var DuplicateVarValid_X1 : String;
  DuplicateVarValid_X1 := 'test';
  Writeln(DuplicateVarValid_X1);
end;

{procedure DuplicateVarInvalid1();
var
  X1 : Integer;
begin
  var X1 : Integer; //Error: Duplicate identifier "X1"
end;}

{procedure DuplicateVarInvalid2();
var
  X1 : Integer;
begin
  begin
    var X1 : Integer; //Error: Duplicate identifier "X1"
  end;
end;}

{procedure DuplicateVarInvalid3();
begin
  var X1 : Integer;
  begin
    var X1 : Integer; //Error: Duplicate identifier "X1"
  end;
end;}

{procedure DuplicateVarInvalid4(X1 : Integer);
begin
  begin
    var X1 : Integer; //Error: Duplicate identifier "X1"
  end;
end;}

procedure TestTypeAssign();
begin
  var X : Integer := 2 + 1;
  var Y : Double := 3.4;
  var Z : String := 'test';
  writeln(X, ' ', Y, ' ', Z);
  begin
    var rec1 : TManagedRecord;
    var rec2 : TManagedRecord := rec1;
  end;
end;

{procedure TestTypeAssignInvalid();
begin
  var X : Integer := 'test';
end;}

procedure TestTypeDetect();
type
  TEnum1 = (Item1, Item2);
  TRec1 = record Data : Integer; end;
begin
  var x1 := 1; //Integer
  var x2 := 2147483648; //Cardinal
  var x3 := -4294967297; //Int64
  var x4 := 9223372036854775808; //Uint64
  var x5 := Word(1); //Integer
  var x6 := Cardinal(1); //Cardinal
  var x7 := Int64(1); //Int64
  var x8 := Default(Word); //Integer
  var x9 := Default(Int64); //Int64
  var e1 := 1.0; //Extended/Double (Inconsistency with DelphiWin64 -> Currency (sic!))
  var e2 := Single(1.0); //Extended/Double anyway
  var e3 := True; //Boolean
  var e4 := LongBool(False); //Boolean anyway
  var e5 := nil; //Pointer
  var a1 := 'c'; //Char
  var a2 := #192; //Char (Inconsistency with Delphi -> AnsiChar) 
  var a3 := #1234; //Char
  var a4 := AnsiChar(#192); //Char (Inconsistency with Delphi -> AnsiChar) 
  var a5 := Char('q'); //Char
  var a6 := Default(AnsiChar); //Char (Inconsistency with Delphi -> AnsiChar)
  var s1 := 'Test1'; //String
  var s2 := ShortString('Test2'); //ShortString
  var s3 := AnsiString('Test3'); //AnsiString
  var s4 := UnicodeString('Test4'); //UnicodeString
  var s5 := WideString('Test5'); //WideString
  var q1 := [1..11]; //Set Of Byte
  var q2 := ['a', 'b', 'c', 'd']; //Dynamic Array of Char
  var q3 := [1,2,3]; //Dynamic Array of Integer
  var q4 := [[1], [2]]; //Dynamic Array of Dynamic Array of Integer (Inconsistency with Delphi -> Internal Error :-] )
  var q5 := TObject.Create; //TObject
  var q6 := TObject(nil); //TObject
  var q7 := TInterfacedObject(nil); //TInterfacedObject
  var q8 := TInterfacedObject.Create; //TInterfacedObject
  var q9 : IUnknown := TInterfacedObject.Create; //IUnknow
  var w1 := Item1; //TEnum1
  var w2 := TEnum1.Item2; //TEnum1
  var w3 := Default(TRec1); //TRec1
  
  //var q101 := [1, 'test']; //compilation error
  //var q102 := []; //compilation error
  //var q103 := [[]]; //compilation error
  //var q104 := [[1, 2], ['test', 'test2']]; //compilation error
  
  //Tests
{  x1 := '';
  x2 := '';
  x3 := '';
  x4 := '';
  x5 := '';
  x6 := '';
  x7 := '';
  x8 := '';
  x9 := '';
  e1 := '';
  e2 := '';
  e3 := '';
  e4 := '';
  e5 := '';
  a1 := '';
  a2 := '';
  a3 := '';
  a4 := '';
  a5 := '';
  s1 := 1;
  s2 := 1;
  s3 := 1;
  s4 := 1;
  s5 := 1;
  q1 := 1;
  q2 := 1;
  q3 := 1;
  q4 := 1;
  q5 := 1;
  q6 := 1;
  q7 := 1;
  q8 := 1;
  q9 := 1;
  w1 := '';
  w2 := '';
  w3 := '';}
end;

{}{}{}

type
  TProc = reference to procedure;
  TProc<T> = reference to procedure(Arg1 : T);
  TFunc<TResult> = reference to function: TResult;

procedure Proc1; begin writeln('Proc1');  end;
procedure Proc2(x1 : Integer); begin writeln('Proc2'); end;
function Func1(par1, par2 : String) : Integer; begin writeln('Func1'); Result := 0; end;
function Func2 : TProc; begin writeln('Func2'); Result := Proc1; end;
function Func3 : TFunc<TProc>; begin writeln('Func3'); Result := Func2; end;
function Func41 : Integer; begin writeln('Func41'); Result := 0; end;
function Func42 : TFunc<Integer>; begin writeln('Func42'); Result := Func41; end;
function Func4 : TFunc<TFunc<Integer>>; begin writeln('Func4'); Result := Func42; end;

type
  TClass1<T> = class
    class procedure D1;
  end;
  
class procedure TClass1<T>.D1;
type
  tn1 = reference to procedure (x1 : T);
begin
  var z1 := 1;
  var n1 : tn1 := procedure (x1 : T) begin 
    writeln('test ref1 ', x1);
    z1 := z1 + 3;
  end;
  n1(Default(T));
  
  {//this isn't working
  var n2 := procedure (x1 : T) begin 
    writeln('test ref1 ', x1);
  end;
  n2(Default(T));
  }
end;

procedure TestTypeDetectRefs();
type
  TRecord1 = record
    a,b,c,d,e,f,g : Integer;
  end;
begin
  var p1 := Proc1; //procedure ()
  //var p2 := Proc1(); //compilation error
  var p3 := Proc2; //procedure (int)
  //var p4 := Proc2(); //compilation error
  var f1 := Func1; //function (string,string):int (Inconsistency with Delphi -> compilation error)
  //var f2 := Func1(); //compilation error
  var f3 := Func1('', ''); //Integer
  //var f4 := Func1(''); //compilation error
  var f5 := Func2; //TProc
  var f6 := Func2(); //TProc
  //var f7 := Func2()(); //compilation error
  var a1 := Func3; //TFunc<TProc>
  var a2 := Func3(); //TFunc<TProc>
  //var a3 := Func3()(); //TProc (runtime error -> FPC bug)
  //var a4 := Func4()()(); //Integer (runtime error -> FPC bug)

  //reference to procedure
  var n1 := function (x1,x3,x4 : Integer; rec1 : TRecord1) : String begin 
    writeln('test ref ', x1,x3,x4);
    result := 'hello';
  end;

  var res := n1(1,2,3,default(TRecord1));
  writeln(res);

  {//Tests
  p1 := '';
  p3 := '';

  f1 := '';
  f3 := '';
  f5 := '';
  f6 := '';
  a1 := '';
  a2 := '';
  a3 := '';
  a4 := '';
  n1 := '';}
  
  TClass1<String>.D1();
end;

procedure TestStatements();
begin 
  //if
  if True then
    var x : Integer
  else
    var x : String;
  if True then
    var y : Integer := 1
  else
    var y : String := 'test';
  if True then
    var y := 1
  else
    var y := 'test';
    
  //case
  case 0 of
    0 : var x : Integer;
    1 : var x := 1;
    2 : var x : String := 'test';
  end;
  
  //repeat
  repeat
    var x := 1;
    var y : String;
  until true;
  //while
  while false do
    var x : String;
  //for
  var ind : Integer;
  for ind := 0 to 1 do
    var x : String;
  for ind in [0..1] do
    var x : String; 
    
  //with
  with TObject.Create do
    var x : String;
  //try
  try
    var x : Integer;
  finally
    var x := 1;
  end;
  try
  except
    var x := 1;
  end;
  try
  except
    on TObject do
      var x := 1;
    else
      var x := 1;
  end;
end;

procedure TestForStatement();
type 
  TEnum1 = (one, two, three, four);
begin
  for var ind := 1 to 4294967295 do 
  begin
    writeln(sizeof(ind)); //4 -> cardinal
    break;
  end;
  for var ind := -1 to 4294967295 do 
  begin
    writeln(sizeof(ind)); //8 -> int64
    break;
  end;
  for var ind := 1 to 18446744073709551615 do 
  begin
    writeln(sizeof(ind)); //8 -> uint64
    break;
  end; 
  for var ind := -1 to 1 do 
  begin
    writeln(sizeof(ind)); //4 -> int
    break;
  end;  
  for var ind := four downto two do 
  begin
    writeln(ind, ' ', sizeof(ind)); //1 -> TEnum1
    break;
  end;
  
  for var ind in 'test' do //string
  begin
    writeln(ind, ' ', SizeOf(ind)); //2 ->Char
    //Break;
  end;
  for var ind in ['a','b','c'] do //array of Char
  begin
    writeln(ind, ' ', SizeOf(ind)); //2 ->Char
    //Break;
  end;
  for var ind in [1,2,3,5] do //array of int
  begin
    writeln(ind, ' ', SizeOf(ind)); //4 ->Int
    //Break;
  end;
  for var ind in [1..3, 6..8] do //set of byte
  begin
    writeln(ind, ' ', SizeOf(ind)); //1 ->byte
    //Break;
  end;
  for var ind in ['a'..'d'] do //set of AnsiChar
  begin
    writeln(ind, ' ', SizeOf(ind)); //1 ->AnsiChar
    //Break;
  end;

{$IfDef DelphiRtl}
  var Dict := TDictionary<String, String>.Create;
  Dict.Add('1', 'one');
  Dict.Add('2', 'two');
  
  //enumerator
  for var item in Dict do
    writeln(item.key, ' = ', item.value);
    
  Dict.Free;
{$EndIf}
end;

procedure TestConsts;
begin
  const x = 1;
  const y : Byte = 2;
  //x := 2; //compilation error
  writeln(x, ' ', SizeOf(x));
  writeln(y, ' ', SizeOf(y));
  
  //Don't do this, just for test
  const Dict = TObject.Create;
  Dict.Free;
end;

begin
  Writeln('ClassicVarOrder');
  ClassicVarOrder;
  Writeln('InlineVarOrder1');
  InlineVarOrder1;
  Writeln('InlineVarOrder2');
  InlineVarOrder2;
  
  Writeln('DuplicateVarValid1');
  DuplicateVarValid1;
  Writeln('DuplicateVarValid2');
  DuplicateVarValid2;
  
  Writeln('TestTypeAssign');
  TestTypeAssign;  
 
  Writeln('TestTypeDetect');
  TestTypeDetect;
  
  Writeln('TestTypeDetectRefs');
  TestTypeDetectRefs;

  Writeln('TestStatements');
  TestStatements;
  
  Writeln('TestForStatement');
  TestForStatement;
  
  Writeln('TestConsts');
  TestConsts;
end.