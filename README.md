require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
import "android.graphics.drawable.GradientDrawable"
import "android.content.*"
import "android.graphics.Typeface"
import "android.net.Uri"

prefs = activity.getSharedPreferences("Dark", Context.MODE_PRIVATE)


function codeToTimestamp(code)
  local letters = "ABCDEFGHJKLMNPQRSTUVWXYZ"
  local num = 0
  for i = 1, #code do
    local c = code:sub(i,i)
    local value = letters:find(c) - 1
    num = num * 23 + value
  end
  return num
end


function isValidKey(inputKey)
  if inputKey == nil or #inputKey < 6 then
    return false, "Invalid License Key"
  end

  local parts = {}
  for part in string.gmatch(inputKey, "[^%-]+") do
    table.insert(parts, part)
  end

  if #parts < 2 then
    return false, "Wrong key format"
  end

  local expiryCode = parts[#parts]
  local expiry = codeToTimestamp(expiryCode)

  if os.time() > expiry then
    return false, "Key Expired"
  end

  return true
end

function showLogin()
  local root = LinearLayout(activity)
  root.setOrientation(1)
  root.setPadding(0,0,0,0)
  root.setGravity(Gravity.CENTER_HORIZONTAL)

  local bgDrawable = GradientDrawable()
  bgDrawable.setColor(0xFF000000)
  bgDrawable.setCornerRadius(0)
  root.setBackground(bgDrawable)

  local title = TextView(activity)
  title.setText("DAVE LOGIN SYSTEM")
  title.setTextColor(0xFFFF4081)
  title.setTextSize(24)
  title.setTypeface(nil, Typeface.BOLD)
  title.setGravity(Gravity.CENTER)
  title.setPadding(0,40,0,20)
  root.addView(title)

  local subtitle = TextView(activity)
  subtitle.setText("Enter License Key to Continue")
  subtitle.setTextColor(0xFF80DEEA)
  subtitle.setTextSize(16)
  subtitle.setGravity(Gravity.CENTER)
  subtitle.setPadding(0,0,0,30)
  root.addView(subtitle)

  local et = EditText(activity)
  et.setHint("Enter key...")
  et.setTextColor(0xFFFFFFFF)
  et.setHintTextColor(0xFF666666)
  et.setTextSize(16)
  local etBg = GradientDrawable()
  etBg.setColor(0xFF111111)
  etBg.setCornerRadius(15)
  etBg.setStroke(2,0xFF333333)
  et.setBackground(etBg)
  et.setPadding(30,20,30,20)
  root.addView(et)

  local cb = CheckBox(activity)
  cb.setText("Remember Me")
  cb.setTextColor(0xFFB2DFDB)
  cb.setTextSize(15)
  cb.setPadding(0,20,0,20)
  root.addView(cb)

  local savedKey = prefs.getString("saved_key", nil)
  if savedKey ~= nil then
    et.setText(savedKey)
    cb.setChecked(true)
  end

  local btnLayout = LinearLayout(activity)
  btnLayout.setOrientation(0)
  btnLayout.setGravity(Gravity.CENTER)
  btnLayout.setPadding(0,20,0,40)

  local ownerBtn = Button(activity)
  ownerBtn.setText("OWNER")
  ownerBtn.setTextColor(0xFFFFFFFF)
  ownerBtn.setTextSize(14)
  local ownerBg = GradientDrawable()
  ownerBg.setColor(0xFFE91E63)
  ownerBg.setCornerRadius(25)
  ownerBtn.setBackground(ownerBg)
  ownerBtn.setPadding(50,20,50,20)
  btnLayout.addView(ownerBtn)

  local space = Space(activity)
  local lp = LinearLayout.LayoutParams(40, 1)
  space.setLayoutParams(lp)
  btnLayout.addView(space)

  local loginBtn = Button(activity)
  loginBtn.setText("LOGIN")
  loginBtn.setTextColor(0xFFFFFFFF)
  loginBtn.setTextSize(14)
  local loginBg = GradientDrawable()
  loginBg.setColor(0xFF00BCD4)
  loginBg.setCornerRadius(25)
  loginBtn.setBackground(loginBg)
  loginBtn.setPadding(60,20,60,20)
  btnLayout.addView(loginBtn)

  root.addView(btnLayout)

  local dialog = AlertDialog.Builder(activity)
  .setView(root)
  .setCancelable(false)
  .create()

  dialog.show()
  dialog.getWindow().setLayout(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT)
  dialog.getWindow().setBackgroundDrawableResource(android.R.color.transparent)

  ownerBtn.onClick=function()
    local url = "https://t.me/AkoSiZayer"
    local intent = Intent(Intent.ACTION_VIEW, Uri.parse(url))
    activity.startActivity(intent)
  end
  loginBtn.onClick=function()
    local input = et.getText().toString()
    local ok,msg = isValidKey(input)
    if ok then
      if cb.isChecked() then
        prefs.edit().putString("saved_key",input).apply()
       else
        prefs.edit().remove("saved_key").apply()
      end
      Toast.makeText(activity,"Login success",Toast.LENGTH_SHORT).show()
      dialog.dismiss()
      showWelcome()
     else
      Toast.makeText(activity,msg,Toast.LENGTH_SHORT).show()
    end
  end
end

showLogin()
