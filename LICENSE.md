package com.example.providers

import com.lagradost.cloudstream3.*
import com.lagradost.cloudstream3.utils.*

class ToonWorld4AllProvider : MainAPI() {

    override var mainUrl = "https://www.toonworld4all.me"
    override var name = "ToonWorld4All"
    override val supportedTypes = setOf(TvType.Anime, TvType.Movie, TvType.TvSeries)

    override val hasMainPage = true

    override val mainPage = listOf(
        MainPageData("Latest", "$mainUrl/", isPaged = true)
    )

    override suspend fun getMainPage(page: Int, request: MainPageRequest): HomePageResponse {
        val doc = app.get(request.data).document
        val items = doc.select(".item-selector").map {
            newSearchResponse(
                it.select(".title-selector").text(),
                it.select("a").attr("href"),
                TvType.TvSeries
            ) {
                posterUrl = it.select("img").attr("src")
            }
        }
        return HomePageResponse(listOf(HomePageList(request.name, items)))
    }

    override suspend fun search(query: String): List<SearchResponse> {
        val doc = app.get("$mainUrl/search?q=$query").document
        return doc.select(".item-selector").map {
            newSearchResponse(
                it.select(".title-selector").text(),
                it.select("a").attr("href"),
                TvType.TvSeries
            ) {
                posterUrl = it.select("img").attr("src")
            }
        }
    }

    override suspend fun load(url: String): LoadResponse {
        val doc = app.get(url).document
        val title = doc.select(".title-page-selector").text()
        return newTvSeriesLoadResponse(title, url, TvType.TvSeries) {
            // parse episodes if available
        }
    }

    override suspend fun loadLinks(
        data: String,
        isCasting: Boolean,
        subtitleCallback: (SubtitleFile) -> Unit,
        callback: (ExtractorLink) -> Unit
    ) {
        loadExtractor(data, subtitleCallback, callback)
    }
}
