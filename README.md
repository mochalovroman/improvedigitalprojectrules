# improve digital project rules

## Работа с develop и production версиями
Для каждого проекта существует production target - <appName> (для примера у нас боевое приложение будет называться "fleksifuotto.fi")
Для каждого проекта существует develop target - <shortAppName-develop> (для примера у нас тестовое приложение будет называться "fleksi-fi-develop")
У каждого обязательно должны быть разные bundle identifier, чтобы:
- в Fabric были разделены production и develop версии
- для утилиты Fastlane которая занимается сборкой проекта и автовыливкой в Fabric, легко работать разными с таргетами
- в разных таргетах можно легко использовать разные константы и идентификаторы (особенно важно чтобы в боевую аналитику не попадали тестовые данные, с таргетами это легко решается)

## Работа с develop версией и генерирование доп. инфы для тестировщика и заказчика на иконке приложения
<img src="http://take.ms/nv99O" alt="Build phases" />
Добавляем в Build Phases секцию проекта следующие скрипты:
- Скрипт для автоинкремена номера билда (инкремент делается для того, чтобы при выливке новой версии в Fabric создавался чистый слот, а это нужно для того чтобы следить за крешами и другими данными по конкретному билду). Обязательно ставим галку на пункте "Run script only when installing", тогда билд будет делать инкремент версии когда будет собираться Архив проекта
```bash
buildNumber=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "$INFOPLIST_FILE")
buildNumber=$(($buildNumber + 1))
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $buildNumber" "$INFOPLIST_FILE"
```
- Скрипт для генерации доп. инфы на иконке приложения
```bash
# номер версии
version=`/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" "${INFOPLIST_FILE}"`
# номер билда
build=`/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "${INFOPLIST_FILE}"`
currentDate=$(date +"%d.%m.%Y")
currentTime=$(date +"%H:%M")

# функция генерации иконки
function processIcon() {
    export PATH=$PATH:/usr/local/bin
    base_file=$1
    target_icon_name=$2
    base_path=`find ${SRCROOT} -name $base_file`
    
    if [[ ! -f ${base_path} || -z ${base_path} ]]; then
    return;
    fi
    
    target_file=`echo $target_icon_name | sed "s/_base//"`
    target_path="${CONFIGURATION_BUILD_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}/${target_file}"
    
    width=`identify -format %w ${base_path}`
    
    echo $target_path
    echo $target_file
    
    convert -background '#0008' -fill white -gravity center -size ${width}x70 -pointsize 18\
    caption:"build - ${build}\nversion - ${version}\n${currentDate}\n${currentTime}"\
    "${base_path}" +swap -gravity south -composite "${target_path}"
}

# запускаем генерацию
processIcon "Icon-120.0.png" "AppIcon60x60@2x.png"
processIcon "Icon-180.0.png" "AppIcon60x60@3x.png"
```
- После установки скрипта для выведения на иконки доп. инфы нужно установить пакеты
```bash
brew install imagemagick
brew install ghostscript
```
А если у вас нет в система Home brew то сходите сюда http://brew.sh/index_ru.html



### Работа с аналитикой и прочими debug/production константами (скоро будет оформлено)
- Работа с develop версией и генерирование доп. инфы для тестировщика и заказчика на иконке приложения
