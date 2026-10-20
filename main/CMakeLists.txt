idf_component_register(SRCS "app_main.c" "board.c" "screen_power.c" "sensors.c" "model.c" "ui.c" "storage.c"
    "vendor/esp_lcd_ili9341.c"
    INCLUDE_DIRS "." "vendor"
    REQUIRES driver esp_lcd esp_timer fatfs sdmmc lvgl)
